# Leaves module included — 20261017-2353 (follow-up)

## Delivered
- **Dedicated map:** `architecture-obecno-leaves-20261017-235727/obecno-leaves.html` → `enhanced/leaves.html`
- **Employee screens** patched + re-finalized with leaves hub / history / types / holidays / sheets
- **Manager screens** patched + re-finalized with Team Leaves / My Leaves / manager detail / status filter
- **screen-catalog.js** — full leave passport entries (screens, sheets, APIs)
- **obecno-api-full-guide.html** — new **Leaves** nav section + expanded employee/manager catalog grids

## Coverage
Employee: ClockLeavesCard → LeavesScreen → history/types/holidays + ApplyLeaveSheet / LeaveDetailSheet / day overview
Manager: Overview Team Leaves / My Leaves → TeamLeavesScreen + ManagerLeaveDetailSheet + filters
APIs: `/employee/leaves*`, `/manager/team-leaves*`

## Note
Manager_leaves UI + LeavesService are partly worktree-uncommitted; diagram topology matches current product surface; pinned revision sources use HEAD-safe paths with `link_mode: local-only`.
