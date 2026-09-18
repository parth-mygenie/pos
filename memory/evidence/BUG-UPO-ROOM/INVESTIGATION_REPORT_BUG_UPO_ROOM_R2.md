# INVESTIGATION REPORT — Room Order Total Bugs (Round 2 — Live Curl Probe)
**Date:** 2026-09-18 (Round 2)
**Evidence path:** `/app/memory/evidence/BUG-UPO-ROOM/`

---

## LIVE PROBE RESULTS — Definitively Confirmed

### API Response Structures (directly curled)

| API | Response contains |
|---|---|
| `PUT update-place-order` | `{"message":"Items added to order successfully!", "order_id":1232501, "total_amount":2098}` **NO order object** |
| `POST order-shifted-room` | `{"message":"Payment Successful via cash"}` **NO order object** |
| `POST get-single-order-new` | Full order with `associated_order_list` ✅ BUT `balance_payment = 2000` (excludes GST) |
| `GET employee-orders-list` | Full order with `associated_order_list` ✅ AND `balance_payment = 2150` (includes GST) |

---

## Bug A: update-place-order → Total drops after adding item to room

### Confirmed Data Flow Break

```
1. FE calls PUT update-place-order
   Response: {"message": ..., "order_id": 1232501, "total_amount": 2098}
   → NO order data → FE ignores response (as designed in OrderEntry.jsx:952)

2. Backend emits socket event: update-order
   Payload: { orders: [{ order_id, order_amount, room_info, ... }] }
   → associated_order_list: ABSENT from socket payload ← ROOT CAUSE

3. handleOrderDataEvent → orderFromAPI.order(payload.orders[0])
   → associated_order_list is undefined → transform produces: associatedOrders = []

4. updateOrder(orderId, transformedOrder) → FULL REPLACE (OrderContext.jsx:144)
   → order.associatedOrders wiped from context

5. computeRoomCardAmount(order):
   food   = 2098  (correct)
   xfer   = 0     ← WRONG (was 874 before socket update)
   roomBal = 2000  ← ALSO WRONG (socket uses raw balance_payment, excludes GST ₹150)
   TOTAL  = 4098  ← Shows wrong total
```

### balance_payment Discrepancy (Bonus Bug)

| Source | balance_payment | Formula |
|---|---|---|
| `employee-orders-list` | **₹2150** | `room_price (3000) + gst (150) − advance (1000) = 2150` |
| `get-single-order-new` | **₹2000** | `room_price (3000) − advance (1000) = 2000` (GST excluded) |
| socket `update-order` | **₹2000** | Same as get-single-order-new (GST excluded) |

The `orderTransform.js:388` reads `api.room_info.balance_payment` → uses ₹2000 from socket, not ₹2150.  
**Even if `associated_order_list` were fixed**, roomBal would still be ₹150 short from socket/get-single-order-new.

---

## Bug B: order-shifted-room → Room card stays stalled

### Confirmed Data Flow Break

```
1. FE calls POST order-shifted-room (source table 1232409 → room r5/1232501)
   Response: {"message": "Payment Successful via cash"}
   → NO room order data whatsoever

2. OrderEntry.jsx:1914-1921:
   removeOrder(sourceOrderId=1232409)      ✅ source removed
   updateTableStatus(sourceTableId, 'available')  ✅ table freed
   // ❌ NOTHING done for room order 1232501

3. Backend emits ONLY for source order:
   update-food-status: [event, 1232409, 69, 6] (status=6 = paid/transferred)
   → handleUpdateFoodStatus: order already removed → skips
   → NO event for room parent order 1232501

4. Room order 1232501 in OrderContext: unchanged
   associatedOrders still = [old 5 entries]  ← STALE (should be 6)
   computeRoomCardAmount = food + 874 + 2150 = STALE (missing new ₹114)
```

### Server IS Updated Immediately — Confirmed

| Check | Before shift | After shift | ∆ |
|---|---|---|---|
| `get-single-order-new` AO count | 5 | **6** | +1 ✅ |
| `get-single-order-new` includes 1232409? | No | **Yes** ✅ | |
| `employee-orders-list` AO count | 5 | **6** | +1 ✅ |
| `employee-orders-list` includes 1232409? | No | **Yes** ✅ | |
| Source table (1232409) in running orders? | Yes | **No** ✅ | removed |

**The server updates immediately.** The gap is purely that the FE never re-fetches the room parent order.

---

## Root Cause Classification

| Bug | Classification | Location |
|---|---|---|
| A — AO stripped in socket | **BACKEND_BUG** | Socket `update-order` payload does not include `associated_order_list` |
| A — balance_payment excludes GST | **BACKEND_BUG** | `get-single-order-new` + socket use raw field; `employee-orders-list` computes correctly |
| B — room parent stale after shift | **FE_GAP** | `OrderEntry.jsx:1903-1937` never re-fetches room parent after `order-shifted-room` |

---

## Recommended Fix Directions

### Bug A — FE-side defensive merge (unblocks without waiting for backend)

In `OrderContext.jsx:130`, merge instead of full-replace for preserved fields:

```js
const updateOrder = useCallback((orderId, updatedOrder) => {
  setOrdersState(prev => {
    const existing = prev.find(o => o.orderId === orderId);
    const merged = existing ? {
      ...updatedOrder,
      // Preserve associatedOrders from existing if socket stripped it
      associatedOrders: (updatedOrder.associatedOrders?.length > 0)
        ? updatedOrder.associatedOrders
        : (existing.associatedOrders || []),
      // Preserve roomInfo — use best available balance_payment
      roomInfo: updatedOrder.roomInfo
        ? {
            ...updatedOrder.roomInfo,
            // If existing had higher balance (GST-inclusive), keep it
            balancePayment: Math.max(
              updatedOrder.roomInfo.balancePayment || 0,
              existing.roomInfo?.balancePayment || 0
            ),
          }
        : (existing.roomInfo || null),
    } : updatedOrder;
    const next = exists
      ? prev.map(o => o.orderId === orderId ? merged : o)
      : [...prev, updatedOrder];
    ordersRef.current = next;
    return next;
  });
}, []);
```

**Risk: HIGH — OrderContext is shared by all orders. Needs careful testing.**

### Bug B — FE re-fetch room parent after shift

In `OrderEntry.jsx:1907`, after successful `order-shifted-room`:

```js
const res = await api.post(API_ENDPOINTS.ORDER_SHIFTED_ROOM, payload);
// ... existing removeOrder + updateTableStatus ...

// NEW: Re-fetch room parent order to update associatedOrders
const roomTableId = paymentData.roomId;
const roomOrders = getOrdersByTableId ? getOrdersByTableId(Number(roomTableId)) : [];
if (roomOrders.length > 0) {
  const roomOrderId = roomOrders[0].orderId;
  const freshRoom = await api.post(API_ENDPOINTS.SINGLE_ORDER_NEW, { order_id: roomOrderId });
  const freshOrders = freshRoom?.data?.orders;
  if (freshOrders?.length > 0) {
    const transformed = orderFromAPI.order(freshOrders[0]);
    updateOrder(roomOrderId, transformed);
  }
}
```

**Note:** This fixes `associatedOrders` but `balance_payment` from `get-single-order-new` = ₹2000 (no GST). Use `employee-orders-list` if balance accuracy needed.

---

## Evidence Files

| File | Contents |
|---|---|
| `update_place_order_response.json` | Confirms response = {message, order_id, total_amount} only |
| `order_shifted_room_response.json` | Confirms response = {message} only |
| `before_shift_single_order.json` | Room r5 state before shift — AO:5 |
| `after_shift_single_order.json` | Room r5 state after shift — AO:6 (server updated immediately) |
| `running_orders_after_shift.json` | employee-orders-list after shift — AO:6, balance_payment=2150 |
| `after_upo_single_order.json` | get-single-order-new after update-place-order |
| `total_comparison.json` | Math comparison across all sources |

---

*Investigation round 2 complete. All root causes confirmed via live curl probes.*
