
# Search Resource Utilization app

This app provides a dashboard that you can use to identify the Splunk searches that use the most resources.

### Search Resource Utilization dashboard

This dashboard relies on the saved search "search resource utilization base", which can be scheduled to run once daily so that the dashboard is highly-responsive without incurring additional search workload.

**View search resources used by app, type, and provenance**
![View search resources used by app, type, and provenance](https://ben-repo-artifacts.s3-us-west-2.amazonaws.com/2020-10-30_13-08-57.png)

**View search resources used over time, and also the label for each search**
![View search resources used over time, and also the label for each search](https://ben-repo-artifacts.s3-us-west-2.amazonaws.com/2020-10-30_13-11-27.png)
After toggling `Run search` to Yes, click the Submit button at the top of the dashboard. This ad-hoc query does not run by default in order to conserve system resources.
