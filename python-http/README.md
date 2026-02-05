# Deliverables

## How to Run Locally

## Python HTTP Track

### Run Service A
```bash
cd service-a
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python app.py
```

### Run Service B (new terminal)
```bash
cd service-b
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python app.py
```

### Test
```bash
curl "http://127.0.0.1:8081/call-echo?msg=hello"
```

Stop Service A and rerun the curl command to observe failure handling.

## Success Proof
{"service_a":{"echo":"hello"},"service_b":"ok"}

## Failure Proof
{"error":"HTTPConnectionPool(host='127.0.0.1', port=8080): Max retries exceeded with url: /echo?msg=hello (Caused by NewConnectionError(\"HTTPConnection(host='127.0.0.1', port=8080): Failed to establish a new connection: [Errno 61] Connection refused\"))","service_a":"unavailable","service_b":"ok"}

## What Makes This Distributed?
This system is distributed because it has two different components (Service A and Service B) that are running as two independent processes and communicating with each other over the network via HTTP. Each Service has its own address, can fail independently, and deals with latency, partial failures, timeouts, etc. It also introduces operational concerns like observability (logs/metrics/traces), independent scaling, and deployment of each service. These factors require distributed-system patterns such as health checks, timeouts/retries, and graceful degradation when dependencies are unavailable.

## What Happens on Timeout?
The requests.get(...,timeout=1.0) in Service B will raise a requests.exceptions.Timeout when Service A does not respond within 1.0 second limit. This is called Timeout and on Timeout, the except block in Service B will be triggered and this makes: Sevice B logs an error message containing the exception string and returns HTTP 503 to the client with error field describing the timeout. 

## What Happens if Service A is Down?
When Service B tries to contact Service A while Service A is down, the requests call inside Service B will raise an exception related to connection error (requests.exceptions.ConnectionError). Service B will catch and log the exception message, then return HTTP 503 and a JSON body indicating the Service A is unvailable.

## What do Your Logs Show, and How Would You Debug?
My logs show:
- timestamp
- service name (A or B)
- sndpoint
- status (ok or error)
- optional error text if status is error
- tatency_ms
For example:
- 2026-02-02 20:12:15,661 service=A endpoint=/echo status=ok latency_ms=0
or 
- 2026-02-02 20:13:05,956 service=B endpoint=/call-echo status=error error="HTTPConnectionPool(host='127.0.0.1', port=8080): Max retries exceeded with url: /echo?msg=hello (Caused by NewConnectionError("HTTPConnection(host='127.0.0.1', port=8080): Failed to establish a new connection: [Errno 61] Connection refused"))" latency_ms=0

To debug:
- Ping curl Service A to confirm if the Service A is up
- Then Curl Service B and check Service B logs to see the connection or timeout error, increase timeout if Service A is slow in response



