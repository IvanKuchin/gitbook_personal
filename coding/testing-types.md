# Testing types

![](../.gitbook/assets/coding\_testing\_types.png)

### Unit testing

Test the smallest units of software. The unit can be a module, function, class, procedure, and so on.

![](../.gitbook/assets/coding\_testing\_unit.png)

### Integration testing

Test how software components work together.

* **Big bang approach:** It requires all components to be finished, and all tests use only actual components.&#x20;
* **Incremental approach:** The component can be tested when it is developed. This approach requires test doubles to replace some dependencies.

&#x20;

### System testing

Test the functionality of the entire system.

* **Functionality testing:** Tests system functions. The focus is on defining system functions and validating that for a given input to the system, you get the desired output.&#x20;
* **Installation testing**: Tests installation procedures and shows if there may be some problems with setting up the system or product on a certain platform, operating system, or device.&#x20;
* **Usability testing**: Tests if the system satisfies requirements for the targeted user base—for example, if the system can be used by visual- or hearing-impaired users (mobile apps with haptic feedback).&#x20;
* **Security testing**: Tests the system for possible security breaches.&#x20;
* **Performance testing**: Tests system performance such as response times, average time to handle request, and number of requests sent in a certain amount of time.&#x20;
* **Load testing**: A subtype of performance testing. The goal is to validate if the system can handle the required load, such as the number of users who are connected at the same time or the minimum number of requests in the queue.&#x20;
* **Stress testing**: A subtype of performance testing. The goal is to validate the stability of a system in extreme conditions such as increased load above the maximum load that the system can handle.&#x20;
* **Regression testing**: Checks if a new version of a system is compatible with older versions.&#x20;
* **Storage testing**: Tests cover usage of Solid State Drives (SSDs) or hard disks—for example, disk storage capacity.&#x20;
* **Configuration testing**: A given system can have different running configurations. An example would be a cloud-based app that uses a configuration file to manage endpoints for used services, which depend on the region and location of the server where the app is deployed. Testing systems against different configurations can help expose fallacies in configuration handling&#x20;
* **Compatibility testing**: Checks if the system is compatible with different environments, operating systems, mobile devices, and so on.&#x20;
* **Reliability testing**: Tests if the same given input system gives the same output. An example of a case where you might get a different result with the same input is a system that delegates the execution of some task dynamically to multiple service instances (microservices) to decrease the load on a single service. When the task is run multiple times, it might be delegated to different services, each of which would return a different result, which would show that this part of the system is not reliable and needs to be fixed.&#x20;
* **Recovery testing**: Tests the system capability of recovery if there is a system failure.&#x20;
* **Procedure testing**: Tests the defined system procedures—for example, the procedure for migrating a database.

&#x20;

### Acceptance testing

Test if the system is ready for delivery.

* **Alpha testing**: Done by developers in a development environment. Developers assume the user role and test the usability of the product.&#x20;
* **Beta testing**: A selected group of users gains access to the product before its release. They use the product and provide feedback, which helps to find missed bugs and increase product quality.&#x20;
* **Contract testing**: Validates that the product satisfies requirements that are requested by a given contract.&#x20;
* **Regulation testing**: Validates that the product satisfies legal and government regulations.&#x20;
* **Operational testing**: Validates that the product is ready for an operational environment. This testing includes validating workflows for client or business operations.

### Performance testing (network)

HTTP performance testing tool.

{% embed url="https://github.com/wg/wrk" %}

### Load testing framework

{% embed url="https://locust.io/" %}

### Stress testing (CPU)

{% embed url="https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource" %}
