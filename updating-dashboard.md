## Add CDVN's Prometheus to the monitoring docker network

## Add a new datasource
- Add a new Prometheus datasource
![Alt text](static/img/datasource1.png?raw=true)
![Alt text](static/img/datasource2.png?raw=true)
- Note down the UID of the datasource
- Use `http://<container name>:9090 in the URL
- ![Alt text](static/img/datasource3.png?raw=true)
- Click "Save & test" button at the bottom

## Updating the Dashboard
### Method 1
- Downlaod and update the dashboard JSON file:
Find sections like this:
```
"datasource": {
  "type": "prometheus",
  "uid": "prometheus"
}
```
Replace the "uid" value with the UID you noted down for the Prometheus datasource.
- Reimport the dashboard

### Method 2
- Add a new Variable for datasource
- ![Alt text](static/img/datasource4.png?raw=true)
- ![Alt text](static/img/datasource6.png?raw=true)
- ![Alt text](static/img/datasource5.png?raw=true)
Now you can either:
- Edit each panel and update the datasource
![Alt text](static/img/datasource7.png?raw=true)
![Alt text](static/img/datasource8.png?raw=true)
- Or Export the dashboard, edit all the relevant sections at once to look like this:
```
"datasource": {
  "type": "prometheus",
  "uid": "${datasource}"
```
