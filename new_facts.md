# Tips for adding a new fact

From time to time, you may need to add a new fact that can be compared. Here is
a rough guide on what to do.

# Background info

Client system data originates from the insights-client tarball that is uploaded
by clients. Once uploaded, it goes from the ingress service to puptoo. Puptoo
then uses insights-core libraries to process it.

After processing, Puptoo creates a message containing a system profile and
sends it to a topic. The inventory service picks the message up from the topic
and saves the updated system profile in the inventory database. It will now
show up in the inventory service (`/api/inventory/`)

After the inventory service saves the record, it emits an event to an egress
queue. The historical system profile archiver picks up the message and saves
the system profile. It is now available under
`/api/historical-system-profiles/` for 7 days.

# checklist for adding a new fact

There are a lot of small steps to add a new fact. It's best to step through
them one by one and confirm things are working after each addition. If you are
adding multiple facts, I recommend adding them one at a time, and to do the
simplest facts (integers, booleans, strings) first. Once you get everything you
want added in, you can then create PRs in various projects.

## insights-client

The very first step is to ensure that insights-client is capturing the data you
need. You can load a tarball and use `insights-core` to inspect it. This is the
library used to parse the tarball by puptoo. For example:

```python
# insert code here to read a tarball and print stuff
```

If you aren't able to read data for the new fact using insights-core, you'll
need to address that before proceeding.

## puptoo

If insights-core is able to read the data, you're in luck! You just have to add
code to puptoo. Here is an example:
https://github.com/RedHatInsights/insights-puptoo/pull/59.

## inventory

Once puptoo is sending the data through, you'll need to tell the inventory
service to save it on the model object. Here is an example of adding a boolean:

```diff
```

To confirm it landed, check that `/api/inventory/v1/openapi.json` has your new
field, and that if you fetch a host via `/api/inventory/v1/hosts/<UUID>`, the
new field is appearing. Note that if the field is not being populated by
puptoo, it will not appear in inventory (unset fields are not shown).

## drift

At this point, you should be able to do an upload with insights-client and see
the uploaded data with the new field in inventory. If you can't, go back to the
prior steps.

To make drift aware of a new field, the first step is to add it to kerlescan.
This is a library used by drift that handles some (but not all) of the profile
parsing. For example, if you are adding a boolean, you'll likely need to add it
here:  https://github.com/beav/kerlescan/blob/master/kerlescan/constants.py#L14

If you are just adding a field, thats it! Drift should be able to pull down the
new field and display it. Run a comparison report using the system you are
testing with and confirm you're able to see the new fact in the drift report.
You can do this either via the web UI or API.
