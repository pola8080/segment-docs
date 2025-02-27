---
title: Destination Filters
rewrite: true
---

Use Destination Filters to prevent certain data from flowing into a destination. With Destination Filters, you can conditionally filter out event properties, traits, and fields, or even filter out the event itself.

Common use cases for Destination Filters include:
- Managing PII (personally identifiable information) by blocking fields from reaching certain destinations
- Controlling event volume by sampling or dropping unnecessary events for specific destinations
- Increasing data relevance in your destinations by removing unused or unwanted data
- Preventing test or internally-generated events from reaching your production tools

> info ""
> Destination Filters are only available to Business Tier customers.

### Limitations

Keep the following limitations in mind when you use Destination Filters:

- Segment applies Destination Filters one at a time in the order that they appear in your workspace.
- Destination Filters can only be applied to cloud-mode (server-side) streaming destinations.
- Device-mode destinations aren't supported.
- You can't apply Destination Filters to Warehouses or S3 destinations.
- Each filter can only apply to one source-destination pair.

[Contact Segment](https://segment.com/help/contact/){:target="_blank"} if these limitations impact your use case.

## Create a Destination Filter

To create a Destination Filter:
1. Go to **Connections > Destinations** and select your destination.
2. Click on the **Filters** tab of your destination.
3. Click **+ New Filter**.
4. Configure the rules for your filter.
5. *(Optional)* Click **Load Sample Event** to see if the event passes through your filter.
6. Click **Next Step**.
7. Name your filter and click the toggle to enable it.
8. Click **Save**.

## Destination Filters API

The Destination Filters API provides more power than Segment's dashboard Destination Filters settings. With the API, you can create complex filters that are conditionally applied using Segment's [Filter Query Language (FQL)](/docs/api/config-api/fql/).

The Destination Filters API offers four different filter types:

| Filter             | Details                                                      |
| ------------------ | ------------------------------------------------------------ |
| `drop_event`       | Doesn't send matched events to the destination.                |
| `sample_event`     | Sends only a percentage of events through to the destination. |
| `whitelist_fields` | Only sends whitelisted properties to the destination.         |
| `blocklist_fields` | Doesn't send blocklisted properties to the destination.        |

To learn more, read Segment's [Destination Filters API docs](https://docs.segmentapis.com/tag/Destination-Filters){:target="_blank"}.

## Examples

The following examples illustrate common Destinations Filters use cases:
* [PII management](#pii-management)
* [Control event volume](#control-event-volume)
* [Cleaner data](#cleaner-data)
* [Remove internal and test events from production tools](#remove-internal-and-test-events-from-production-tools)
* [Sample a percentage of events](#sample-a-percentage-of-events)
* [Drop events](#drop-events)  

### PII management

Example: Remove email addresses from `context` and `properties`:

Property-level allowlisting is available with Segment's API. Using Destination Filters, you can configure a rule that removes email addresses from `context` and `properties`. As a result, Segment only sends traits without PII to the destination.


![PII management example](images/destination-filters/pii_example.png)

### Control event volume

This example shows a filter that controls event volume by only sending `User Signed Up` and `Demo Requested` events.

![Example of a filter that controls event volume](images/destination-filters/drop_example.png)

### Cleaner data

This example shows a rule that only sends track calls to Google Analytics.

![Example of a filter that only sends track calls to Google Analytics](images/destination-filters/clean_example.png)

### Remove internal and test events from production tools

In the example below, the rule targets email addresses with internal domains to stop test events from reaching Destinations.

![Example of a filter that removes internal and test events from production tools](images/destination-filters/internal_example.png)

In the example below, the rule prevents an event from sending if `Order Completed` and `properties.email` contain an internal `@segment.com` email address.

![Internal domain filter example](images/destination-filters/internal_example2.png)

### Sample a percentage of events

Using the [Destination Filters API](https://docs.segmentapis.com/tag/Destination-Filters){:target="_blank"}, you can create a rule to randomly sample video heartbeat events.

### Drop events

[Watch this Destination Filters walkthrough](https://www.youtube.com/watch?v=47dhAF1Hoco){:target="_blank"} to learn how to use event names to filter events sent to destinations.

## Important notes

#### Conflicting settings

Some destinations offer settings that also allow you to filter data. For example, the Facebook App Events destination allows you to map `Screen` events to `Track` events. Because Destination Filters are evaluated and applied _before_ the Destination settings are applied, they can conflict with your settings.

For example, if you have a Destination Filter that filters Track events _and_ you have the **Use Screen Events as Track Events** setting enabled, `Track` events drop, but `Screen` events still process. The destination settings transform it into a `Track` event - *after* the filters.

#### Error handling

Segment makes effort to ensure that Destination Filters can handle unexpected situations. For example, if you use the `contains()` FQL function on the `null` field, Segment returns `false` instead of returning an error. If Segment can't infer your intent, Segment logs an internal error and drops the event. Segment defaults to this behavior to prevent sensitive information, like a PII filter, from getting through.

Errors aren't exposed in your Destination's Event Deliverability tab. For help diagnosing missing destination filter events, [contact Segment](https://segment.com/help/contact/){:target="_blank"}.

## FAQs

#### How do Destination Filters work with array properties?

Destination Filters can filter properties out of objects nested in an array. For example, you can filter out the `price` property of every object in an array at `properties.products`. You can also filter out an entire array from the payload. However, you can't drop nested objects in an array or filter properties out of a single object in an array.

To block a specific property from all of the objects within a properties array, set the filter using the following the format: `<propertyType>.<arrayName>.<arrayElementLabel>​`.

For example, the `properties.products.newElement` filter blocks all `newElement` property fields from each `products` object of an array within the `properties` object of a Track event.

![Filter array properties](images/destination-filters/filter-array-properties.png)

To block the Identify event trait `products.newElement`, select the option under the **User Traits** list instead. To block the context object field `products.newElement`, select it from the **Context Fields** list.

#### How many filters can I create?

Segment supports 10 filters per destination. If you need help consolidating filters or would like to discuss your use case, [contact Segment](https://segment.com/help/contact/){:target="_blank"}.

#### Can I set multiple `Only Send` destination filters?

Segment evaluates multiple `Only Send` filters against each other and resolves Destination Filters in order. If multiple `Only Send` filters conflict with each other, Segment won't send information downstream.

#### How many properties can I view in the filter dropdown?

Segment displays the most recent 15,000 properties. To find a property not in the filter dropdown, enter the property manually.

#### How can I filter out warehouse events?

To filter out events from warehouses, use Selective Sync.

#### I don't see a *name* property at the top level of my events to filter on *event* name".

Generally, only Track calls have *name* properties, which correspond to the *event* field in an event.

#### How can I find out when new Destination Filters have been added or removed?

The Activity Feed shows the action, date, and user who performed the action when a Destination Filter is created, modified, enabled, disabled, or deleted. You can also subscribe to notifications for any of these changes in the **Activity Feed** settings page.

#### Why am I getting a permissions denied error when I try to save a filter?

You must have write access to save and edit filters. Read permission access only allows viewing and testing access.

#### How can I test my filter?

Use the Destination Filter tester during setup to verify that you're filtering out the right events. Filtered events show up on the schema page but aren't counted in event deliverability graphs.
