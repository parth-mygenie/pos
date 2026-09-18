# BACKEND BRIEF — Room Order Real-Time Total Bugs
**Date:** 2026-09-18  
**Prepared by:** FE Investigation (Role 6 — INVESTIGATION)  
**Test account:** owner@thegoankitchen.com | restaurant_id: 69  
**Test order:** 1232501 | #000090 | Room r5 | customer: boiler  
**Evidence path:** `/app/memory/evidence/BUG-UPO-ROOM/`  
**Priority:** P1 HIGH  

---

## Confirmation: Both Bugs Are Backend Socket Emission Gaps

The FE already has all the correct socket handlers wired and working.  
**Zero FE changes needed** — once backend sends the right socket payload, both bugs are fixed.

---

## BUG A — `update-place-order` causes room card total to drop

### What Happens Today (Broken)

**Step 1 — Cashier adds item to room order via POS**

```
PUT /api/v2/vendoremployee/order/update-place-order
Response: {"message": "Items added to order successfully!", "order_id": 1232501, "total_amount": 2435}
```
FE correctly ignores this response body (by design — socket is source of truth).

**Step 2 — Backend emits socket event**

```
Channel : new_order_69
Payload : ["update-order", 1232501, 69, 5, { "orders": [ <order_object> ] }]
```

**The problem is what's INSIDE `order_object`:**

```json
{
  "id": 1232501,
  "order_amount": 2435,
  "room_info": {
    "balance_payment": "2000.00",
    "room_price": "3000.00",
    "gst_tax": "150.00",
    "advance_payment": "1000.00"
  }
  // ❌ associated_order_list : KEY IS ABSENT FROM PAYLOAD
  // ❌ balance_payment       : 2000 (raw), should be 2150 (room_price + gst_tax - advance)
}
```

**Step 3 — FE transforms socket payload (`orderFromAPI.order(payload.orders[0])`)**

```
api.associated_order_list = undefined
  → transform produces: order.associatedOrders = []    ← WIPED (was 8 items = ₹1101)

api.room_info.balance_payment = 2000
  → transform produces: order.roomInfo.balancePayment = 2000  ← WRONG (should be 2150)
```

**Step 4 — FE does full replace in OrderContext**

```
updateOrder(1232501, transformedOrder)  → replaces entire order object in memory
```

**Step 5 — Card computes wrong total**

```
computeRoomCardAmount(order):
  food    = 2435   ✅ (correct)
  xfer    = 0      ❌ (was ₹1101 from 8 associated orders — now wiped)
  roomBal = 2000   ❌ (should be ₹2150 — GST ₹150 missing)
  CARD    = ₹4435  ← USER SEES THIS (WRONG)
```

**Step 6 — After page reload**

FE calls `employee-orders-list` which returns the full correct data:

```
  food    = 2434.95
  xfer    = 1101     ✅ (8 associated orders present)
  roomBal = 2150     ✅ (GST-inclusive balance)
  CARD    = ₹5685.95 ← CORRECT
```

---

### What Backend Needs to Send (Fixed)

Include `associated_order_list` in socket payload AND fix `balance_payment` formula:

```json
Channel: new_order_69
Payload: ["update-order", 1232501, 69, 5, {
  "orders": [{
    "id": 1232501,
    "order_amount": 2435,

    "associated_order_list": [
      { "id": 1232502, "restaurant_order_id": "000091", "order_amount": 114, "collect_Bill": "2026-09-18 09:27:22" },
      { "id": 1232504, "restaurant_order_id": "000092", "order_amount": 209, "collect_Bill": "..." },
      { "id": 1232507, "restaurant_order_id": "000093", "order_amount": 114, "collect_Bill": "..." },
      { "id": 1232511, "restaurant_order_id": "000094", "order_amount": 209, "collect_Bill": "..." },
      { "id": 1232512, "restaurant_order_id": "000095", "order_amount": 114, "collect_Bill": "..." },
      { "id": 1232409, "restaurant_order_id": "000070", "order_amount": 114, "collect_Bill": "..." },
      { "id": 1232514, "restaurant_order_id": "000096", "order_amount": 114, "collect_Bill": "..." },
      { "id": 1232515, "restaurant_order_id": "000097", "order_amount": 113, "collect_Bill": "..." }
    ],

    "room_info": {
      "room_price": "3000.00",
      "gst_tax": "150.00",
      "advance_payment": "1000.00",
      "balance_payment": 2150,
      "room_payment_summary": {
        "remaining_room_balance": 2150
      }
    },

    ...all other existing fields unchanged...
  }]
}]
```

**Minimum `associated_order_list` item fields required by FE:**

```json
{ "id": <int>, "restaurant_order_id": "<string>", "order_amount": <number>, "collect_Bill": "<datetime>" }
```

**`balance_payment` correct formula:**

```
balance_payment = room_price + gst_tax - advance_payment
               = 3000 + 150 - 1000
               = 2150   ← what employee-orders-list already returns (correct)
```

`get-single-order-new` currently uses `room_price - advance` (= 2000, missing GST). The socket payload must match `employee-orders-list` logic.

---

### Exact Gap Summary for Bug A

| What | Currently in socket payload | What is needed |
|---|---|---|
| `associated_order_list` | **ABSENT** (key missing) | Present — all transferred order entries |
| `room_info.balance_payment` | `"2000.00"` (raw, no GST) | `2150` (room_price + gst_tax − advance) |
| `room_payment_summary.remaining_room_balance` | `2000` | `2150` |

---

---

## BUG B — `order-shifted-room` room card total stays stalled

### What Happens Today (Broken)

**Step 1 — Cashier shifts table order to room**

```
POST /api/v2/vendoremployee/order/order-shifted-room
Body: { "order_id": "1232402", "room_id": "8527", ... }

Response: {"message": "Payment Successful via cash"}
```
No order data in response at all.

**Step 2 — FE does optimistic cleanup immediately**

```
removeOrder(1232402)               ← source table order removed from FE context
updateTableStatus(8531, 'available') ← source table freed
navigateAfterOrderAction()           ← returns to dashboard
```

**Step 3 — Backend emits socket event**

```
Channel: new_order_69
Socket : ["update-food-status", 1232402, 69, 6]   ← source order, status=6 (paid)
```

**Step 4 — FE socket handler tries to process it**

```javascript
// socketHandlers.js line 401
if (getOrderById && !getOrderById(orderId)) {
  log('INFO', `update-food-status: Order already removed, skipping`);
  return;   ← EXITS immediately, does nothing
}
```

`getOrderById(1232402)` returns `null` because FE already removed it in Step 2.  
Handler exits early. **No API call is made. Room parent is never touched.**

**Step 5 — Room r5 (1232501) in FE context is STALE**

```
order.associatedOrders = [8 old items]   ← stale (should be 9, missing 1232402)
computeRoomCardAmount  = food + old_xfer + roomBal  ← shows old total
```

**Server IS updated immediately — confirmed by curl probe right after the API call:**

```
get-single-order-new(1232501): AO count = 9 ✅ (was 8, now includes 1232402)
employee-orders-list(1232501): AO count = 9 ✅ (was 8, now includes 1232402)
Table 1 (1232402) no longer in running orders: True ✅
```

**Server has the right data. FE context is never told.**

**Step 6 — Only page reload fixes it**

After reload, `employee-orders-list` returns fresh data with AO=9 and the correct card total.

---

### What Backend Needs to Send (Fixed)

After `order-shifted-room` succeeds, **additionally emit** an `update-order` event for the **ROOM PARENT ORDER** (not the source order):

```
Channel: new_order_69
Payload: ["update-order", 1232501, 69, 5, {
  "orders": [{
    "id": 1232501,
    "order_amount": 2434.95,

    "associated_order_list": [
      { "id": 1232502, "restaurant_order_id": "000091", "order_amount": 114,  "collect_Bill": "..." },
      { "id": 1232504, "restaurant_order_id": "000092", "order_amount": 209,  "collect_Bill": "..." },
      { "id": 1232507, "restaurant_order_id": "000093", "order_amount": 114,  "collect_Bill": "..." },
      { "id": 1232511, "restaurant_order_id": "000094", "order_amount": 209,  "collect_Bill": "..." },
      { "id": 1232512, "restaurant_order_id": "000095", "order_amount": 114,  "collect_Bill": "..." },
      { "id": 1232409, "restaurant_order_id": "000070", "order_amount": 114,  "collect_Bill": "..." },
      { "id": 1232514, "restaurant_order_id": "000096", "order_amount": 114,  "collect_Bill": "..." },
      { "id": 1232515, "restaurant_order_id": "000097", "order_amount": 113,  "collect_Bill": "..." },
      { "id": 1232402, "restaurant_order_id": "000064", "order_amount": 107,  "collect_Bill": "<now>" }
      ← NEW entry (just-shifted order)
    ],

    "room_info": {
      "balance_payment": 2150,
      "room_payment_summary": { "remaining_room_balance": 2150 }
    },

    ...all other room order fields unchanged...
  }]
}]
```

The FE's existing `handleOrderDataEvent` handler will process this `update-order` event, transform it, call `updateOrder(1232501, ...)` and the room card total will update instantly.

---

### Exact Gap Summary for Bug B

| What | Currently happens | What is needed |
|---|---|---|
| Socket event for source order (1232402) | `update-food-status` fired | Existing — fine as-is |
| Socket event for room parent (1232501) | **NOTHING emitted** | `update-order` with full payload including updated `associated_order_list` |

---

## Summary of 3 Required Backend Changes

| # | Trigger API | Change | Impact |
|---|---|---|---|
| 1 | `update-place-order` | Include `associated_order_list` in `update-order` socket payload for RM orders | Fixes Bug A — AO wipe |
| 2 | `update-place-order` | Fix `balance_payment` = `room_price + gst_tax - advance` in socket + `get-single-order-new` | Fixes Bug A — ₹150 GST gap |
| 3 | `order-shifted-room` | After shift, emit `update-order` for the **room parent order** with updated `associated_order_list` | Fixes Bug B — stale card |

All three changes are **socket emission layer only**. Server data is already correct. No DB changes. No new endpoints.

---

## FE Handler Reference (Already Working — No Changes Needed)

```
useSocketEvents.js:127-128
  case 'update-order':
    handleOrderDataEvent(args, actionsRef.current, 'update-order')

socketHandlers.js handleOrderDataEvent:
  1. Reads  payload.orders[0]                   ← backend must populate this correctly
  2. Calls  orderFromAPI.order(payload.orders[0]) ← reads associated_order_list + room_info
  3. Calls  updateOrder(orderId, transformed)    ← updates card in real-time
```

Once backend sends correct payload, the card total updates live.  
No FE code changes required.

---

## Curl Evidence (Reproduced This Session)

| API Call | Response | Key Observation |
|---|---|---|
| `PUT update-place-order` | `{"message":..., "order_id":1232501, "total_amount":2435}` | No order object in response |
| `POST order-shifted-room` | `{"message":"Payment Successful via cash"}` | No order object in response |
| `POST get-single-order-new` (after update-place-order) | AO=8 ✅, balance_payment=2000 ❌ | AO present, balance wrong |
| `GET employee-orders-list` (after update-place-order) | AO=8 ✅, balance_payment=2150 ✅ | Both correct |
| `POST get-single-order-new` (after shift) | AO=9 ✅ includes 1232402, balance_payment=2000 ❌ | AO updated, balance wrong |
| `GET employee-orders-list` (after shift) | AO=9 ✅ includes 1232402, balance_payment=2150 ✅ | Both correct |
