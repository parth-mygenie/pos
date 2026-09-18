# INVESTIGATION REPORT — Room Order Total Bugs
**Date:** 2026-09-18  
**Role:** INVESTIGATION  
**Scope:** Two bugs on room order totals after API calls  
**Steps used:** 9/10  
**Evidence path:** `/app/memory/evidence/BUG-UPO-ROOM/`

---

## 1. Summary

### Bug A — update-place-order: Checkout total drops (₹3476→₹3326) after adding item to room order
- **Root cause:** `update-order` socket payload strips `associated_order_list` → `associatedOrders = []` post-update
- **Classification:** BACKEND_BUG (socket payload incomplete) + DATA_ISSUE (balance_payment discrepancy between two APIs)
- **Confidence:** HIGH (reproduced via data comparison + math checks out exactly)
- **Steps used:** 1–7

### Bug B — order-shifted-room: Room card amount stalled (no socket update for parent room order)
- **Root cause:** Backend emits no socket event for the ROOM parent order after `order-shifted-room`; FE only removes source order, never refreshes room order's `associatedOrders`
- **Classification:** BACKEND_BUG (missing socket event for room parent)
- **Confidence:** HIGH (code trace confirms no socket event path exists for room parent update)
- **Steps used:** 8–9

---

## 2. Order Under Test

| Field | Value |
|---|---|
| order_id | 1232501 |
| restaurant_order_id | #000090 |
| Room | r5 |
| Customer | boiler |
| restaurant_id | 69 (The Goan Kitchen) |

---

## 3. Bug A — Detailed Trace

### Data Flow

```
DashboardPage.computeRoomCardAmount(order):
  food   = order.amount               → sourced from api.order_amount
  xfer   = sum(order.associatedOrders[].amount)  → sourced from api.associated_order_list[]
  roomBal = order.roomInfo.balancePayment         → sourced from api.room_info.balance_payment
  return food + xfer + roomBal
```

### API Comparison (CONFIRMED)

| Source | order_amount | associated_order_list | room_info.balance_payment | computeRoomCardAmount |
|---|---|---|---|---|
| `employee-orders-list` (boot/reload) | ₹1325.1 | 3 orders = ₹437 | **₹2150** (computed remaining) | **₹3912.1** ✅ |
| `get-single-order-new` (POST probe) | ₹1326 | 3 orders = ₹437 | **₹2000** (raw booking field) | ₹3763 |
| socket `update-order` payload (inferred from observed UI) | ₹1326 | **EMPTY []** (stripped) | ₹2000 (raw booking field) | **₹3326** ❌ |

### Math Verification
- **Observed after update:** ₹3326
- **Reverse-engineered:** 1326 (food) + **0** (transfers, stripped) + 2000 (raw balance) = **3326** ✅ exact match
- **After reload:** ₹3912.1 = 1325.1 + 437 + 2150 (from employee-orders-list) ✅ exact match

### Root Cause
The backend's `update-order` socket event payload (emitted after `update-place-order`) does **not** include `associated_order_list`. The FE's `handleOrderDataEvent` transforms `payload.orders[0]` directly → `orderFromAPI.order()` → `associatedOrders = []` (because `api.associated_order_list` is undefined/empty in socket payload).

`updateOrder()` in `OrderContext.jsx:144` does a **full replace** (`prev.map(o => o.orderId === orderId ? updatedOrder : o)`), so the previously-correct `associatedOrders` from boot is wiped.

Secondary issue: `room_info.balance_payment` in socket payload = raw booking amount (₹2000) vs `employee-orders-list` computed remaining (₹2150). This is a separate discrepancy in `balance_payment` field logic between the two backend endpoints.

### Break Point in Data Flow
```
API → transform (orderFromAPI.order) → socket update-order payload
       ↑ api.associated_order_list = MISSING in socket payload
       ↓ order.associatedOrders = []   (lost!)
OrderContext.updateOrder(orderId, transformedOrder)  → full replace
computeRoomCardAmount → food + 0 + 2000 = 3326 ❌
```

---

## 4. Bug B — Detailed Trace

### Code Path: order-shifted-room

```
OrderEntry.jsx:1904-1937:
  api.post(ORDER_SHIFTED_ROOM, payload)           // POST room transfer
  .then(res => {
    removeOrder(sourceOrderId)                    // ✅ source removed from context
    updateTableStatus(sourceTableId, 'available') // ✅ source table freed
    navigateAfterOrderAction()                    // → back to dashboard
  })
  // NO fetch/update for the ROOM parent order 🚫
```

### Socket Events After order-shifted-room

Backend emits after `order-shifted-room`:
- `update-food-status` for the SOURCE order → FE handler (`handleUpdateFoodStatus`) sees order is gone, skips
- **NO event** for the ROOM PARENT order to update its `associated_order_list`

Socket event list on FE (`socketEvents.js`):
```
UPDATE_ORDER, UPDATE_ORDER_TARGET, UPDATE_ORDER_SOURCE, UPDATE_ORDER_PAID,
UPDATE_FOOD_STATUS, UPDATE_ORDER_STATUS, SCAN_NEW_ORDER, UPDATE_TABLE, ORDER_ENGAGE
```
None of these are triggered with a fresh `associated_order_list` for the ROOM order.

### Break Point in Data Flow
```
order-shifted-room POST success:
  source order → removeOrder() ✅
  room order → UNCHANGED in OrderContext ❌
    room.associatedOrders still = [old list, if any]
    computeRoomCardAmount → shows stale total

employee-orders-list (on reload):
  room order → fresh data with new entry in associated_order_list ✅
    computeRoomCardAmount → correct total
```

---

## 5. Evidence Artifacts

| File | Content |
|---|---|
| `login_response.json` | Auth response |
| `running_orders.json` | `employee-orders-list` — confirmed `associated_order_list` + `room_info` present |
| `single_order_new_1232501.json` | `get-single-order-new` — `associated_order_list` present, `balance_payment=2000` |
| `total_comparison.json` | Math comparison across 3 data sources |

---

## 6. Recommendations

### Bug A — Fix Options

**Option 1 (Backend — preferred):** Ensure `update-order` socket event payload includes `associated_order_list` for room orders. Also align `room_info.balance_payment` to use the computed remaining amount (from `room_payment_summary.remaining_room_balance`).

**Option 2 (Frontend workaround):** In `updateOrder()` (`OrderContext.jsx:130`), merge instead of replace for room orders:
```js
// If incoming update lacks associatedOrders, preserve from existing
const merged = {
  ...updatedOrder,
  associatedOrders: updatedOrder.associatedOrders?.length
    ? updatedOrder.associatedOrders
    : (existing.associatedOrders || []),
  roomInfo: updatedOrder.roomInfo ?? existing.roomInfo,
};
next = prev.map(o => o.orderId === orderId ? merged : o);
```
Risk: MEDIUM — touches OrderContext which is a hotspot.

**Option 3 (Frontend workaround):** After `update-place-order` succeeds + socket `update-order` fires, fetch `get-single-order-new` for room orders and call `updateOrder` with fresh data. This is heavier but safer.

### Bug B — Fix Options

**Option 1 (Backend — preferred):** After `order-shifted-room`, also emit an `update-order` socket event for the ROOM PARENT order with updated `associated_order_list`.

**Option 2 (Frontend workaround):** After successful `order-shifted-room` POST (`OrderEntry.jsx:1907`), extract the room order ID (from payload `room_id` → `getOrderByTableId`), then fetch `get-single-order-new` for room order and call `updateOrder(roomOrderId, freshData)`.
```js
// After line 1907: const res = await api.post(ORDER_SHIFTED_ROOM, payload)
const roomTableId = paymentData.roomId; // tableId of room
const roomOrder = getOrderByTableId(roomTableId)?.[0];
if (roomOrder) {
  const fresh = await api.post(API_ENDPOINTS.SINGLE_ORDER_NEW, { order_id: roomOrder.orderId });
  const transformed = orderFromAPI.order(fresh.data.orders[0]);
  updateOrder(roomOrder.orderId, transformed);
}
```
Risk: MEDIUM — requires `getOrderByTableId` in collect-bill context.

---

## 7. Planning Skip Eligibility

### Bug A
- Touches 2 files minimum (OrderContext.jsx + possibly socketHandlers.js) → **NOT eligible** for fast lane
- Risk: **HIGH** (order amount display, socket logic, room billing)
- Recommended: Full gate cycle (Gate 2-3) → PLANNING role

### Bug B
- Touches OrderEntry.jsx (hotspot file per R5) → **NOT eligible** for fast lane
- Risk: **HIGH** (room transfer flow, financial total display)
- Recommended: Full gate cycle (Gate 2-3) → PLANNING role

---

## 8. Backend Briefs Needed

Both bugs have backend-side root causes. Backend briefs should be raised for:

1. `update-order` socket payload — add `associated_order_list` for room orders
2. `room_info.balance_payment` — align to `room_payment_summary.remaining_room_balance` in socket/single-order endpoints
3. `order-shifted-room` — emit socket event for ROOM PARENT with updated `associated_order_list`

---

*Investigation complete. Steps used: 9/10.*
