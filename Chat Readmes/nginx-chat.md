## Step-by-Step Guide to Helm Chart for Nginx

1. **Create a namespace**
    Create a namespace in your cluster within which this work will be done
    ```bash
    kubectl create ns savvyminds
    ```

2. **Create the Nginx Helm Chart**

   In your Kubernetes directory, create a Helm chart specifically for Nginx:

   ```bash
   helm create nginx-chart
   ```

   This will generate a new folder named `nginx-chart` with a standard Helm structure:
   
   ```
   nginx-chart/
   ├── charts/
   ├── templates/
   │   ├── deployment.yaml
   │   ├── service.yaml
   │   └── ...
   ├── Chart.yaml
   └── values.yaml
   ```

**steps 3, 4 and 5 are not compulsory, you can igonore the editings and just look through**

3. **Customize `values.yaml`**

   Open the `values.yaml` file and edit it to reflect the configurations for your Nginx deployment:

   ```yaml
   replicaCount: 1

   image:
     repository: nginx
     tag: latest
     pullPolicy: IfNotPresent

   service:
     type: ClusterIP
     port: 80

   ingress:
     enabled: false

   resources: {}
   
   nodeSelector: {}
   tolerations: []
   affinity: {}
   ```

4. **Edit the Deployment Template**

   Go to `templates/deployment.yaml` and customize it to match the Nginx deployment:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: {{ .Release.Name }}-nginx
     labels:
       app: {{ .Release.Name }}-nginx
   spec:
     replicas: {{ .Values.replicaCount }}
     selector:
       matchLabels:
         app: {{ .Release.Name }}-nginx
     template:
       metadata:
         labels:
           app: {{ .Release.Name }}-nginx
       spec:
         containers:
         - name: nginx
           image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
           ports:
           - containerPort: 80
   ```

5. **Edit the Service Template**

   Modify the `templates/service.yaml` file to create the ClusterIP service for Nginx:

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: {{ .Release.Name }}-nginx-service
     labels:
       app: {{ .Release.Name }}-nginx
   spec:
     type: {{ .Values.service.type }}
     ports:
     - port: {{ .Values.service.port }}
       targetPort: 80
     selector:
       app: {{ .Release.Name }}-nginx
   ```

6. **Deploy the Chart**

   With the Helm chart customized, you can now deploy it to your Kubernetes cluster:

   ```bash
   helm install nginx-release ./nginx-chart -n savvyminds
   ```

   This command installs the chart with the release name `nginx-release` in the `savvyminds` namespace.

7. **Confirm creation of pods and services**
    
    use the following commands to confirm the creation of the pods and services
    
    ```bash
    kubectl get pod -n savvyminds
    ```

    ```bash
    kubectl get services -n savvyminds
    ```

8. **Upgrade and Update the Chart**

   If you make changes to the `values.yaml` or templates, update the deployment with:

   ```bash
   helm upgrade nginx-release ./nginx-chart -n savvyminds
   ```

9. **Port-Forward the Service (Optional)**

   Use port-forwarding if you want to access the Nginx service locally:

   ```bash
   kubectl port-forward service/nginx-release-nginx-chart 8080:80 -n savvyminds
   ```

10. **Uninstall the Chart**

   To remove the deployment, use:

   ```bash
   helm uninstall nginx-release -n savvyminds
   ```

### Helm Chart Structure Summary

- **`Chart.yaml`**: Metadata about the Helm chart, including its name and version.
- **`values.yaml`**: Default values for your templates (like image version and replica count).
- **`templates/`**: Contains your Kubernetes resource templates, where you use Helm's templating syntax.

