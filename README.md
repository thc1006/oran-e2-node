# O-RAN E2 Node Simulator

**Author**: 蔡秀吉 (thc1006)
**Version**: 1.0.0
**Last Updated**: 2025-11-15

## Overview

This repository contains an E2 Node Simulator designed to test O-RAN RIC Platform xApps by generating realistic E2 interface traffic. The simulator supports multiple xApps and can be configured to simulate various network scenarios.

## Features

- **Multi-xApp Support**: Simultaneously sends E2 indications to:
  - KPIMON (port 8081)
  - Traffic Steering (port 8081)
  - QoE Predictor (port 8090)
  - RAN Control (port 8100)
  - Federated Learning (port 8110)

- **Realistic Data Generation**: Simulates KPIs including:
  - PRB utilization (DL/UL)
  - Active UE count
  - Throughput metrics
  - Latency measurements
  - RSRP/RSRQ values
  - CQI/MCS values

- **Configurable Simulation**:
  - Adjustable iteration interval (default: 30s)
  - Cell ID configuration (default: 1234567)
  - Extensible xApp target configuration

## Architecture

```
┌─────────────────────┐
│  E2 Node Simulator  │
│  (Containerized)    │
└──────────┬──────────┘
           │
           │ HTTP POST /e2/indication
           │
           ├──────────┬──────────┬──────────┬──────────┐
           ↓          ↓          ↓          ↓          ↓
     ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐
     │ KPIMON  │ │ Traffic │ │   QoE   │ │   RAN   │ │Federated│
     │  :8081  │ │Steering │ │Predictor│ │ Control │ │Learning │
     │         │ │  :8081  │ │  :8090  │ │  :8100  │ │  :8110  │
     └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘
```

## Quick Start

### Prerequisites

- Kubernetes cluster (k3s or other)
- kubectl configured
- Docker (for building custom images)
- O-RAN RIC Platform deployed in `ricxapp` namespace

### Deployment

```bash
# Deploy using kubectl
kubectl apply -f deploy/deployment.yaml

# Verify deployment
kubectl get pods -n ricxapp | grep e2-simulator

# Check logs
kubectl logs -n ricxapp -l app=e2-simulator -f
```

### Building Custom Image

```bash
# Build Docker image
docker build -t localhost:5000/e2-simulator:1.0.0 .

# Push to local registry
docker push localhost:5000/e2-simulator:1.0.0

# Update deployment
kubectl rollout restart deployment/e2-simulator -n ricxapp
```

## Configuration

### xApp Targets

Edit `src/e2_simulator.py` to configure xApp targets:

```python
self.xapp_targets = {
    'kpimon': {
        'host': 'kpimon.ricxapp.svc.cluster.local',
        'port': 8081,
        'endpoint': '/e2/indication'
    },
    # Add more xApps...
}
```

### Simulation Parameters

```python
# Iteration interval (seconds)
iteration_interval = 30

# Cell ID
cell_id = 1234567
```

## Monitoring

The simulator logs all activities:

```bash
# View simulation iterations
kubectl logs -n ricxapp -l app=e2-simulator | grep "Simulation Iteration"

# View connection status
kubectl logs -n ricxapp -l app=e2-simulator | grep "Connection"

# View error messages
kubectl logs -n ricxapp -l app=e2-simulator | grep -i "error\|failed"
```

## Integration with RIC Platform

This E2 Node Simulator is designed to work with the [O-RAN RIC Platform](https://github.com/thc1006/oran-ric-platform) project. It enables:

- **xApp Development**: Test xApp logic without real RAN equipment
- **Metrics Validation**: Verify Prometheus metrics integration
- **Performance Testing**: Simulate various network conditions
- **CI/CD Integration**: Automated testing in deployment pipelines

## Extending the Simulator

### Adding New xApp Targets

1. Update `self.xapp_targets` in `src/e2_simulator.py`
2. Configure correct service name, port, and endpoint
3. Rebuild and redeploy

### Adding New KPIs

1. Update `generate_kpi_data()` method
2. Add new metrics to the data structure
3. Ensure receiving xApp can process the new fields

### Custom Scenarios

Create scenario-specific data generators:

```python
def generate_high_load_scenario(self):
    """Simulate high network load"""
    return {
        'prb_usage_dl': random.uniform(80, 100),
        'prb_usage_ul': random.uniform(70, 90),
        'active_ue_count': random.randint(100, 200)
    }
```

## Troubleshooting

### Simulator Not Sending Data

**Check**:
- xApp services are running: `kubectl get svc -n ricxapp`
- Network connectivity: `kubectl exec -n ricxapp e2-simulator-xxx -- curl -I http://kpimon.ricxapp.svc.cluster.local:8081`

**Solution**:
- Verify service names and ports match configuration
- Check xApp readiness probes are passing

### Connection Errors

**Symptom**: `Connection error for <xapp> (xApp may not have REST endpoint yet)`

**Solution**:
- Ensure xApp has implemented `/e2/indication` endpoint
- Verify xApp Service exposes correct port
- Check xApp Pod is in `Running` state

### Data Not Reaching xApps

**Check**:
- xApp logs: `kubectl logs -n ricxapp <xapp-pod-name>`
- Prometheus metrics: Check if counters are incrementing

**Solution**:
- Verify xApp endpoint logic is correct
- Check xApp metrics implementation

## Technical Details

### Dependencies

- Python 3.9+
- requests
- Kubernetes service discovery (DNS)

### Docker Image

Base image: `python:3.9-slim`

Size: ~150MB

### Resource Requirements

```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "50m"
  limits:
    memory: "128Mi"
    cpu: "100m"
```

## Contributing

This simulator is part of the O-RAN RIC Platform deployment project. For issues, enhancements, or questions:

1. Check existing documentation in the main platform repository
2. Create an issue with detailed description
3. Follow the project's coding standards

## Related Projects

- [O-RAN RIC Platform](https://github.com/thc1006/oran-ric-platform) - Main platform repository
- [O-RAN SC](https://wiki.o-ran-sc.org/) - O-RAN Software Community

## License

This project follows the same license as the O-RAN SC RIC Platform.

## Changelog

### Version 1.0.0 (2025-11-15)
- Initial release
- Support for 5 xApps (KPIMON, Traffic Steering, QoE, RC, FL)
- Configurable simulation parameters
- Comprehensive KPI data generation
- Kubernetes-native deployment

---

**Maintained by**: 蔡秀吉 (thc1006)
**Repository**: https://github.com/thc1006/oran-e2-node
