(This is a draft guide only. It is missing many details, use it at your own risk)

## 1. Add a new datasource
Add the CDVN's Prometheus container to the monitoring docker network (if CDVN and monitoring stack are on the same machine)  
If the monitoring stack is on a nexternal machine, then think about port mapping (firewall it) or a tunnel (e.g. traefik)
- Add a new Prometheus datasource  
![Alt text](static/img/datasource1.png?raw=true)
![Alt text](static/img/datasource2.png?raw=true)
- Note down the UID of the datasource (from the browser URL section)  
- Use `http://<container name>:9090` in the server URL  
![Alt text](static/img/datasource3.png?raw=true)
- Click "Save & test" button at the bottom  

## 2. Updating the Dashboard
### 2.1 Method 1 (work best with a single datasource)
- Download and update the dashboard JSON file (charon_overview_dashboard.json): [Link](https://github.com/ObolNetwork/charon-distributed-validator-node/tree/main/grafana/dashboards)  
- Find sections like this:
```
"datasource": {
  "type": "prometheus",
  "uid": "prometheus"
}
```
- Replace the "uid" value with the UID you noted down for the Prometheus datasource  
- (Re-)import the dashboard to Grafana  

### 2.2 Method 2 (enable switching between multiple datasources)
- Add a new Variable for datasource  
![Alt text](static/img/datasource4.png?raw=true)
![Alt text](static/img/datasource6.png?raw=true)
![Alt text](static/img/datasource5.png?raw=true)
Now you can either:  
- Edit each panel one by one and update the datasource  
![Alt text](static/img/datasource7.png?raw=true)
![Alt text](static/img/datasource8.png?raw=true)
- Or Export the dashboard, edit all the relevant sections at once to look like this:  
```
"datasource": {
  "type": "prometheus",
  "uid": "${datasource}"
```
