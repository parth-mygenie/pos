# BACKEND_BRIEF — Room Order Socket Gaps
**Date:** 2026-09-18  
**From:** FE Investigation Agent  
**Priority:** P1 — HIGH  
**Bugs covered:** BUG-UPO-ROOM-A + BUG-UPO-ROOM-B  
**Evidence path:** `/app/memory/evidence/BUG-UPO-ROOM/`

---

## Confirmation: Both Bugs Are Backend Socket Gaps

Both bugs are exclusively backend issues. The FE already has the correct handlers in place — `handleOrderDataEvent` processes `update-order` events and calls `updateOrder(orderId, order)` which updates the dashboard card in real time. The FE needs NO changes if the backend sends the correct socket payloads.

| Bug | Where the gap is | FE side | Backend side |
|---|---|---|---|
| A — Total drops after `update-place-order` | Socket `update-order` payload is incomplete | Handler exists, works correctly ✅ | Payload missing `associated_order_list` + wrong `balance_payment` ❌ |
| B — Room card stalls after `order-shifted-room` | No socket event emitted for room parent | Handler exists, works correctly ✅ | No event emitted for room parent order ❌ |

---

## Gap 1 — `update-order` Socket Payload is Incomplete for Room Orders

### Trigger
`PUT /api/v2/vendoremployee/order/update-place-order`

### What backend currently emits
```
Channel: new_order_{restaurant_id}
Message: [
  "update-order",           // [0] event name
  1232501,                  // [1] order_id
  69,                       // [2] restaurant_id
  5,                        // [3] f_order_status
  {                         // [4] payload
    "orders": [
      {
        "id": 1232501,
        "order_amount": 2098,
        "room_info": { "balance_payment": "2000.00", ... },
        // ❌ associated_order_list: ABSENT
        // ❌ balance_payment: 2000 (raw, excludes GST)
        ...other fields...
      }
    ]
  }
]
```

### What backend must emit
```
Channel: new_order_{restaurant_id}
Message: [
  "update-order",
  1232501,
  69,
  5,
  {
    "orders": [
      {
        "id": 1232501,
        "order_amount": 2098,
        // ✅ GAP 1A — ADD this field for room orders (order_in = 'RM')
        "associated_order_list": [
          {
            "id": 1232502,
            "restaurant_order_id": "000091",
            "order_amount": 114,
            "collect_Bill": "2026-09-18 09:27:22"
          },
          ...all transferred orders
        ],
        // ✅ GAP 1B — balance_payment must include GST
        "room_info": {
          "room_price": "3000.00",
          "advance_payment": "1000.00",
          "gst_tax": "150.00",
          "balance_payment": 2150,          // ← room_price + gst_tax - advance_payment
          "room_payment_summary": {
            "remaining_room_balance": 2150   // ← same value
          },
          ...other existing fields
        },
        ...other fields
      }
    ]
  }
]
```

### Proof of Impact

The FE computes the room card amount as:
```
computeRoomCardAmount(order) =
  order.amount                                    // food
  + sum(order.associatedOrders[].amount)         // transferred table orders
  + order.roomInfo.balancePayment                // room booking balance
```

| State | food | transfers | roomBal | Card shows |
|---|---|---|---|---|
| After socket (current — broken) | ₹1326 | ₹0 (AO stripped) | ₹2000 (no GST) | **₹3326** ❌ |
| After socket (fixed) | ₹1326 | ₹437 | ₹2150 | **₹3913** ✅ |
| After page reload (correct) | ₹1325.1 | ₹437 | ₹2150 | **₹3912.1** ✅ |

### `associated_order_list` minimum fields required by FE transform
```json
{
  "id": 1232502,
  "restaurant_order_id": "000091",
  "order_amount": 114,
  "collect_Bill": "2026-09-18 09:27:22"
}
```
The FE maps: `id → orderId`, `order_amount → amount`, `collect_Bill → transferredAt`.  
Richer shape from `get-single-order-new` (with `waiter_name`, `item_names`, etc.) is also accepted — minimum is the 4-field shape.

### `balance_payment` fix
The `employee-orders-list` endpoint already returns the correct value:  
`balance_payment = room_price + gst_tax − advance_payment = 3000 + 150 − 1000 = 2150`  
The `get-single-order-new` endpoint uses `room_price − advance_payment = 2000` (no GST).  
**The socket payload must match `employee-orders-list` logic — include GST in `balance_payment`.**

---

## Gap 2 — No Socket Event for Room Parent After `order-shifted-room`

### Trigger
`POST /api/v2/vendoremployee/order/order-shifted-room`

### What backend currently emits
```
After order-shifted-room:

1. Channel: new_order_69
   Message: ["update-food-status", 1232409, 69, 6]   ← source order (table) status=6 (paid/transferred)

2. ❌ NOTHING emitted for room parent order 1232501
```

### What backend must additionally emit
```
After order-shifted-room, ALSO emit on the same channel:

Channel: new_order_{restaurant_id}
Message: [
  "update-order",           // ← existing event name, already handled by FE
  1232501,                  // ← room PARENT order_id (not the source order)
  69,                       // ← restaurant_id
  5,                        // ← f_order_status of room parent (unchanged, 5 = running)
  {
    "orders": [
      {
        // Full room parent order object WITH updated associated_order_list
        "id": 1232501,
        "order_amount": 1984.5,
        "associated_order_list": [
          { "id": 1232502, "restaurant_order_id": "000091", "order_amount": 114, "collect_Bill": "..." },
          { "id": 1232504, "restaurant_order_id": "000092", "order_amount": 209, "collect_Bill": "..." },
          { "id": 1232507, "restaurant_order_id": "000093", "order_amount": 114, "collect_Bill": "..." },
          { "id": 1232511, "restaurant_order_id": "000094", "order_amount": 209, "collect_Bill": "..." },
          { "id": 1232512, "restaurant_order_id": "000095", "order_amount": 114, "collect_Bill": "..." },
          { "id": 1232409, "restaurant_order_id": "000070", "order_amount": 114, "collect_Bill": "..." }
          // ← NEW entry — the just-shifted order
        ],
        "room_info": {
          "balance_payment": 2150,   // ← must include GST (same fix as Gap 1B)
          ...
        },
        ...all other standard order fields
      }
    ]
  }
]
```

### Proof that server data is ready immediately

Curled `get-single-order-new` for room 1232501 immediately after `order-shifted-room` call — server already has the updated `associated_order_list`:

```
Before shift:  associated_order_list count = 5
After shift:   associated_order_list count = 6  ← 1232409 present immediately
```

The server-side data IS updated. The only missing piece is the backend broadcasting it to connected clients via socket.

### Why `update-food-status` for source order is not enough
The FE `handleUpdateFoodStatus` checks if the source order (1232409) exists in context — it does not (it was already removed optimistically by FE), so the handler skips. Even if it did process it, `update-food-status` carries only `[event, orderId, restaurantId, status]` — no order payload, and no room parent info.

---

## Summary of Required Backend Changes

| # | API | Change | Urgency |
|---|---|---|---|
| 1 | `update-place-order` socket emission | Include `associated_order_list` in `update-order` payload for room orders | **P1** |
| 2 | `update-place-order` socket emission | Use GST-inclusive `balance_payment` in `room_info` (= room_price + gst_tax − advance) | **P1** |
| 3 | `order-shifted-room` socket emission | After shift completes, emit `update-order` on `new_order_{rid}` channel for the room PARENT order | **P1** |
| 4 | `order-shifted-room` socket emission | Same `update-order` payload must also include `associated_order_list` + GST-inclusive `balance_payment` | **P1** |

All four changes are on the **socket emission layer only**. No API response format changes, no DB changes, no new endpoints. The data already exists server-side.

---

## FE Socket Handler — Confirmation It Is Already Wired

The FE handles `update-order` on `new_order_{restaurantId}` channel already:

```
useSocketEvents.js:127-128:
  case SOCKET_EVENTS.UPDATE_ORDER:
    handleOrderDataEvent(args, actionsRef.current, 'update-order');

socketHandlers.js:238-344:
  handleOrderDataEvent:
    → transforms payload.orders[0] via orderFromAPI.order()
    → calls updateOrder(orderId, order)    ← updates card immediately
    → calls syncTableStatus(order, ...)   ← updates table state
```

**Once backend includes the correct fields, the FE will display the correct total with zero FE changes.**

---

## Evidence

| File | What it proves |
|---|---|
| `update_place_order_response.json` | API response = `{message, order_id, total_amount}` — no order object |
| `order_shifted_room_response.json` | API response = `{message}` — no order object |
| `before_shift_single_order.json` | Room state before shift: AO=5 |
| `after_shift_single_order.json` | Room state after shift via `get-single-order-new`: AO=6 immediately ✅ |
| `running_orders_after_shift.json` | Room state after shift via `employee-orders-list`: AO=6, balance_payment=2150 |
| `total_comparison.json` | Math: 1326+0+2000=3326 (broken) vs 1326+437+2150=3913 (correct) |
