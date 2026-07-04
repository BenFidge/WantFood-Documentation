# Recalculate ratings

You use this workflow when the platform needs to recompute vendor, branch, or dish ratings after review changes or maintenance work.

## When you use this feature
- moderation has changed review visibility
- ratings look stale or incorrect
- support or operations need to refresh rating calculations

## Where the workflow starts
Flagged reviews list or other rating-related admin context.

## How it works
1. Identify why rating maintenance is needed.
2. Trigger the rating recalculation workflow.
3. Allow the background process to complete.
4. Recheck the visible rating state after normal propagation.

## Expected result
Displayed ratings catch up with the current review state.

## Related
- [Flagged reviews list](../pages/flagged-reviews-list.md)
- [Automation and background jobs](../../cross-cutting/automation-and-background-jobs.md)
- [Propagation and cache behaviour](../../cross-cutting/propagation-and-cache-behaviour.md)

