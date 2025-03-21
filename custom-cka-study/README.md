# CKA Practice Questions

<br/>

## q1: ETCD Backup and Restore

Perform ETCD backup and restore scenario on node `cluster1-controlplane`. Use the following information:

ETCD Endpoint: https://127.0.0.1:2379
CA Certificate: /etc/kubernetes/pki/etcd/ca.crt
Server Certificate: /etc/kubernetes/pki/etcd/server.crt
Server Key: /etc/kubernetes/pki/etcd/server.key

Tasks:
1. Backup current ETCD data to `/opt/backup/etcd-backup.db` using the certificates provided
2. Create a test namespace called `pre-restore` and deployment `nginx` with image `nginx:1.22`
3. Restore ETCD data to backup point
4. Verify test resources (namespace and deployment) are gone after restore

<br/>

## q2: Multi-Container Pod Troubleshooting

Create and troubleshoot a multi-container pod with the following requirements:

### Initial Setup
1. Create a pod named `multi-container-pod` with two containers:
   - Container 1: nginx (name: nginx)
   - Container 2: busybox (name: log-monitor)

2. Configure shared volumes:
   - Volume `shared-data`: mounted at `/usr/share/nginx/html` in nginx container
   - Volume `nginx-logs`: mounted at `/var/log/nginx` in both containers

3. Configure log monitoring:
   - `busybox` container should continuously monitor nginx access logs
   - Use command: `tail -f /var/log/nginx/access.log`

### Troubleshooting Tasks
1. Fix permission issues:
   - Ensure both containers can access nginx log files
   - Configure appropriate security context
   - Verify log monitoring is working

### Verification Steps
1. Verify pod is running:
```bash
kubectl get pod multi-container-pod
```

2. Test nginx is working:
```bash
kubectl exec multi-container-pod -c nginx -- curl localhost
```

3. Check log monitoring:
```bash
kubectl logs multi-container-pod -c log-monitor
```

4. Verify shared volumes:
```bash
kubectl exec multi-container-pod -c nginx -- ls /usr/share/nginx/html
kubectl exec multi-container-pod -c log-monitor -- ls /var/log/nginx
```

Note: All containers should run with appropriate permissions to access shared resources.

<br/>

## q3: Node Maintenance
Perform maintenance on one of the worker nodes:
1. Mark node as unschedulable
2. Safely evacuate running pods to other nodes
3. Simulate system upgrade (wait 30 seconds)
4. Mark node as schedulable again

<br/>

## q4: Configure Network Policy
Configure a network policy that meets the following requirements:

## Initial Settings
1. Create the following namespaces:
- Front end
- Backend
- Database

2. Distribute test nginx pods to each namespace:
- front end/front end-pot
- Backend/Backend-Pod
- Database/DB-Pod

## Network Policy Requirements
1. Frontend Namespace:
- Can only communicate with the pad in the backend namespace
- Block all other ingress/egress traffic
- Label: Role = Front End

2. backend Namespace:
- Allow traffic from the frontend
- Allow only outgoing traffic to database namespaces
- Label: Role = Backend

3. Database Namespace:
- Allow traffic from the backend only
- Blocking All Aggress Traffic
- Label: Role=database

## Verification Test
Perform the following tests to ensure that the policy is applied correctly:

1. Front-end -> Back-end Communication Test
2. Frontend -> Check Database Communication Blocking
3. Backend -> Database Communication Test
4. database -> confirmation of external communication block

The test is performed using the following command:
```bash
kubectl execute -it <pod-name> -n <namespace> -- wget -qO- -- timeout=2 http://<target-pod-ip>
```

Record all communication test results in the file '/opt/network-policy-test.txt'.


<br/>

## q5: User authentication and permission management

Configure certificate-based authentication for new development team administrators.

### Initial Settings
1. Create a new user certificate with the following information:
- Username: developer-admin
- Group: devops-team
- Certificate validity: 365 days
- Certificate storage location: `/opt/certificates/developer-admin.crt`
- Private key storage location: `/opt/certificates/developer-admin.key`

2. Create a `development` name space and grant the following privileges:
- All rights to Deployment, Pod, Service resources
- Read permissions for ConfigMap and Secret
- No access to Namespace resources

### kubeconfig settings
1. Create a kubeconfig file with the following settings:
- File location: `/opt/certificates/developer-admin.kubeconfig`
- Context name: `developer-context`
- Cluster name: `kubernetes`
- Namespace: `development`

### Test permissions
Make sure your permissions are set correctly by performing the following tasks:

1. Test authentication with the new kubeconfig:
```bash
kubectl --kubeconfig=/opt/certificates/developer-admin.kubeconfig get pods -n development
```

2. Test the following privileges and record the results to /opt/developer-admin-tests.txt:
- Whether Deployment can be created
- ConfigMap Readability
- Secret Readability
- Can't create Namespace

### Estimated deliverables
1. Certificate files:
- /opt/certificates/developer-admin.crt
- /opt/certificates/developer-admin.key
2. kubeconfig file:
- /opt/certificates/developer-admin.kubeconfig
3. Test Results File:
- /opt/developer-admin-tests.txt

Note: All certificates must be signed with the cluster's CA, and files must be generated with the appropriate permissions.

<br/>

## q6: Resource Management and Scheduling
Configure pod scheduling with following requirements:
1. Schedule specific pods only on nodes with GPU label
2. Distribute memory-intensive pods across specific nodes
3. Set anti-affinity rules for certain pods
4. Configure node resource usage monitoring

<br/>

## q7: Service and Ingress Configuration
Perform service configuration with following requirements:
1. Expose application using NodePort service
2. Set up ingress controller
3. Configure domain-based routing rules
4. Apply SSL/TLS certificates

<br/>

## q8: Storage Configuration
Configure storage with following requirements:
1. Create StorageClass for dynamic provisioning
2. Deploy StatefulSet using PVC
3. Create and restore volume snapshots
4. Test storage expansion

<br/>

## q9: Logging and Monitoring
Configure cluster monitoring system:
1. Install and configure metrics-server
2. Set up HorizontalPodAutoscaler
3. Verify node and pod metrics collection
4. Test resource usage based auto-scaling

<br/>

## q10: Cluster Upgrade
Perform cluster upgrade procedure:
1. Plan upgrade from current version to next minor version
2. Upgrade control plane components
3. Sequentially upgrade worker nodes
4. Test functionality after upgrade

<br/>

## q11: CoreDNS Configuration
Update CoreDNS configuration:
1. Backup current CoreDNS configuration
2. Add custom domain resolution
3. Configure stub domain
4. Verify DNS resolution works correctly

<br/>

## q12: Pod Security Policy
Implement pod security policies:
1. Create policy preventing privileged containers
2. Configure volume type restrictions
3. Set up container user and group restrictions
4. Test policy enforcement

<br/>

## q13: Cluster Troubleshooting
Diagnose and resolve cluster issues:
1. Fix broken kubelet service
2. Resolve API server connectivity issues
3. Debug pod scheduling failures
4. Repair broken service networking

<br/>

## q14: Custom Resource Definition
Work with custom resources:
1. Create CRD for application configuration
2. Implement custom controller
3. Test resource validation
4. Verify custom resource functionality

<br/>

## q15: High Availability Configuration
Configure high availability:
1. Set up multi-master control plane
2. Configure etcd cluster
3. Implement load balancing
4. Test failover scenarios

Note: Each question is designed to test practical skills required for the CKA exam and real-world Kubernetes administration.
