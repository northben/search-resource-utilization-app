
# Search Resource Utilization app

This app provides a dashboard that you can use to identify the Splunk searches that use the most resources.

### Search Resource Utilization dashboard

This dashboard relies on the saved search "search resource utilization base", which can be scheduled to run once daily so that the dashboard is highly-responsive without incurring additional search workload.

**View search resources used by app, type, and provenance**
![View search resources used by app, type, and provenance](https://ben-repo-artifacts.s3-us-west-2.amazonaws.com/2020-10-30_13-08-57.png)

**View search resources used over time, and also the label for each search**
![View search resources used over time, and also the label for each search](https://ben-repo-artifacts.s3-us-west-2.amazonaws.com/2020-10-30_13-11-27.png)
After toggling `Run search` to Yes, click the Submit button at the top of the dashboard. This ad-hoc query does not run by default in order to conserve system resources.

### Query Explanation

**This query is used as the basis for the dashboard.**

Search the _introspection index for events with search IDs (sid) but exclude real-time and system searches:
```
index=_introspection sourcetype=splunk_resource_usage data.search_props.sid::* data.search_props.mode!=RT data.search_props.user!="splunk-system-user"
```

Extract some fields, and calculate an identifiable label for the search job:
```
| eval elapsed = 'data.elapsed'
| eval mem_used = 'data.mem_used'
| eval read_mb = 'data.read_mb'
| eval sid = 'data.search_props.sid'
| eval app = 'data.search_props.app'
| eval type = 'data.search_props.type'
| eval mode = 'data.search_props.mode'
| eval user = 'data.search_props.user'
| eval role = 'data.search_props.role'
| eval label = 'data.search_props.label'
| eval label = if(isnull(label) AND match(sid, ".*_subsearch_.*"), "subsearch", label)
| eval label = if(isnull(label) AND match(sid, ".*__(search\d+)_.*"), "dashboard panel", label)
| eval label = if(isnull(label) AND match(label, ".*_ACCELERATE_.*"), "acceleration", label)
| eval label = if(isnull(label) AND type=="ad-hoc","ad-hoc", label)
```

Calculate the provenance, and originating search head/peer:
```
| eval provenance = if(isnotnull('data.search_props.provenance'), 'data.search_props.provenance', 'data.search_props.role')
| eval search_head = case(isnotnull('data.search_props.search_head') AND 'data.search_props.role' == "peer", 'data.search_props.search_head', isnull('data.search_props.search_head') AND 'data.search_props.role' == "head", "_self", isnull('data.search_props.search_head') AND 'data.search_props.role' == "peer", "_unknown")
```

Selecting just the fields we need from here makes it easy to work with the query, then perform the first stats which aggregates to a single result for each sid:
```
| fields _time label provenance type mode app role user elapsed mem_used read_mb search_head sid
| stats count dc(user) earliest(_time) as _time values(label) values(provenance) values(type) values(mode) values(app) values(role) values(user) sum(elapsed) sum(mem_used) sum(read_mb) values(search_head) by sid
```

Rename the stats fields all at once:
```
| rename values(*) as *, sum(*) as sum_*
```

Finally, summarize the individual searches by day, app, type, and provenance:
```
| bin _time span=1d
| stats sum(sum_elapsed) as sum_elapsed sum(sum_mem_used) as sum_mem_used sum(sum_read_mb) as sum_read_mb by _time app type provenance
```
