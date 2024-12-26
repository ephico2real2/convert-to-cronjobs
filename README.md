# convert-to-cronjobs

### **Summary of Research and Plan for Converting Helm Chart Deployment to CronJob**

#### **Background and Context**
The Jira story involves converting a Helm chart template for a Kubernetes `Deployment` to a `CronJob`. This change is necessary to enable periodic execution of workloads that were previously running continuously. Since this task involves Helm charts, the templates may include placeholders (`{{ .Values }}`), conditionals, and other templating logic, which adds complexity to the conversion process.

---

### **Advantages of Using `CronJob` Over `Deployment` for Long-Running Processes**

1. **Efficient Resource Utilization**:
   - **`CronJob`**: Runs tasks only at specified intervals, freeing up cluster resources when the job is not executing. This is ideal for processes that don’t need to run continuously, like periodic data synchronization, backups, or cleanup tasks.
   - **`Deployment`**: Keeps resources like Pods and their replicas running at all times, even when not actively needed, leading to potential wastage.

2. **Simplified Scheduling**:
   - **`CronJob`**: Native support for scheduling jobs using a `schedule` field (in CRON format), eliminating the need for custom scheduling logic in the application.
   - **`Deployment`**: Requires additional tools like external schedulers or manual application-level logic to achieve similar functionality.

3. **Reduced Complexity**:
   - **`CronJob`**: Removes the need to manage long-running Pods for tasks that are inherently periodic. Pods are created dynamically and terminated after task completion.
   - **`Deployment`**: Maintains Pods continuously, even for tasks that only need to execute periodically, requiring manual intervention to stop and restart tasks.

4. **Built-in Retry and Failure Handling**:
   - **`CronJob`**: Automatically retries failed jobs according to the configured `.spec.backoffLimit`, ensuring reliability without additional setup.
   - **`Deployment`**: Relies on application-level retry mechanisms, increasing complexity for error handling.

5. **No Need for PodDisruptionBudgets (PDBs)**:
   - **`CronJob`**: As Pods are transient and only exist during task execution, PDBs are unnecessary, simplifying the configuration.
   - **`Deployment`**: Requires careful management of PDBs to ensure availability during updates or disruptions.

6. **Cost Optimization**:
   - **`CronJob`**: By running only when required, CronJobs reduce compute costs, making them a cost-effective solution for periodic tasks.
   - **`Deployment`**: Continuously running Pods incur costs even when idle.

7. **Native Kubernetes Support**:
   - **`CronJob`**: Leverages native Kubernetes functionality for task scheduling and execution, requiring no additional components or external schedulers.
   - **`Deployment`**: Needs extensions or manual scripting to achieve similar periodic execution behavior.

---

### **Research Summary**

1. **Understanding the Difference**:
   - **Deployment**: Manages long-running applications with replica management, rolling updates, and optional PodDisruptionBudgets (PDBs).
   - **CronJob**: Schedules tasks to run periodically or at specific times, eliminating the need for continuous replication or PDBs.

2. **Challenges Identified**:
   - **Converting Deployment-Specific Fields**:  
     - Removing fields like `replicas`, `strategy`, and PDBs, which are not relevant to CronJobs.
   - **Restructuring YAML**:  
     - Transforming `spec.template` to align with the `jobTemplate.spec` structure required by CronJobs.
   - **Helm Templating Compatibility**:  
     - Ensuring the Helm chart structure remains intact, including placeholders and conditionals.
   - **Adding Schedule Field**:  
     - Incorporating a `schedule` field dynamically into the Helm template (e.g., `{{ .Values.cron.schedule }}`) for periodic job execution.

3. **Options for Implementation**:
   - **Manual Conversion**:
     - Directly editing YAML files for small-scale use cases or one-off requirements.
   - **Automation Using Scripts**:
     - **Shell Script**:  
       - Use `sed` for quick transformations of rendered YAML files, suitable for simple, flat structures.
     - **Python Script**:  
       - Use `pyyaml` to handle complex, deeply nested YAML and enable future scalability for templated Helm charts.

---

### **Plan to Address the Jira Story**

1. **Proposed Approach**:
   - Focus on automating the conversion process with a **Python script** for structured parsing and transformation of YAML files.
     - Replace `kind: Deployment` with `kind: CronJob`.
     - Restructure `spec.template` into `jobTemplate.spec`.
     - Add a `schedule` field using Helm templating placeholders (e.g., `{{ .Values.cron.schedule }}`).
     - Remove unsupported fields like `replicas`, `strategy`, and PDB references.
   - Maintain Helm templating logic in the converted YAML, ensuring compatibility with `.Values` placeholders and other Helm constructs.

2. **Validation Process**:
   - Use Helm tools to validate the updated chart:
     - `helm lint` to check the chart structure for errors.
     - `helm template` to render the templates with specific `values.yaml` files for verification.
   - Validate the rendered CronJob YAML with Kubernetes dry-run:
     ```bash
     kubectl apply --dry-run=client -f rendered-cronjob.yaml
     ```
   - Deploy the converted CronJob in a sandbox environment to test periodic execution.

3. **Documentation**:
   - Document the conversion process, including how to use the Python script and adapt Helm templates for CronJob-specific configurations.
   - Provide clear examples of modifying `values.yaml` to specify schedules and other CronJob parameters.

4. **Future Scalability**:
   - Ensure the solution supports multiple environments (`staging`, `production`) by parameterizing environment-specific fields in `values.yaml`.

---

### **Progress Update**
- Research completed on the structural differences between `Deployment` and `CronJob`.
- Challenges and considerations for Helm templating compatibility have been identified.
- Validation tools and steps (e.g., `helm lint`, `kubectl dry-run`) have been outlined.
- A clear plan has been defined to handle conversion, validation, and documentation.
