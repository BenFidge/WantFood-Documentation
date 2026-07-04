# Activate or deactivate vendor

You use this workflow to control whether a vendor can participate in active platform workflows.

## When you use this feature
- a vendor should return to live use
- a vendor should be taken out of live use
- the current vendor state no longer matches the business decision

## Where the workflow starts
Vendors list -> Vendor details.

## How it works
1. Open the vendor record.
2. Review the current status and confirm the target state.
3. Run the activation or deactivation action.
4. Check any downstream visibility or propagation behaviour that matters to the business.

## Expected result
The vendor moves into the correct active or inactive state and downstream behaviour can catch up through normal propagation.

## Related
- [Vendor details](../pages/vendor-details.md)
- [Propagation and cache behaviour](../../cross-cutting/propagation-and-cache-behaviour.md)

