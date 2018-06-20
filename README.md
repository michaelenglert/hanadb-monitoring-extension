# AppDynamics SAP HanaDB Monitoring Extension

This extension works only with the standalone machine agent.

## Use Case

Deployable on premise or in the cloud, SAP HANA is an in-memory data platform that lets you accelerate business processes, deliver more business intelligence, and simplify your IT environment. By providing the foundation for all your data needs, SAP HANA removes the burden of maintaining separate legacy systems and siloed data, so you can run live and make better business decisions in the new digital economy. HanaDB Monitoring Extension gathers metrics and sends them to the AppDynamics Metric Browser.

## Prerequisites

 * This extension requires the Java Machine Agent.
 * The HanaDB Driver `ngdbc.jar` is required
 * A HanaDB User with the following **privileges** is required:
   * **MONITORING:** this role contains privileges for full read-only access to all metadata, the current system status in system and monitoring views, and the data collected by the statistics server
   * **PUBLIC:** this role contains privileges for filtered read-only access to the system views

## Installation

Either [Download the Extension from the AppDynamics Marketplace](https://www.appdynamics.com/community/exchange/hanadb-monitoring-extension/), [Download the Extension from the Github releases](https://github.com/michaelenglert/hanadb-monitoring-extension/releases/latest) or Build from Source.

1. Deploy the `HanaDBMonitor-<VERSION>.zip` file into the `<machine agent home>/monitors` directory.

  `> unzip HanaDBMonitor-<VERSION>.zip -d <machine agent home>/monitors/`

2. Copy the `ngdbc.jar` in the `<machine agent home>/monitors/HanaDBMonitor/` folder

3. Set up `config.yml`. At minimum this is:

  ```
  clusters:
    - connectionString: "jdbc:sap://<cluster1_host1>:<port>;<cluster1_host2>:<port>?communicationtimeout=20000"
      username: ""
      password: ""
  ```

3. Restart the Machine Agent.

## Build from Source

1. Clone this repository
2. Run `mvn -DskipTests clean install`
3. The `HanaDBMonitor-<VERSION>.zip` file can be found in the `target` directory

## Directory Structure

<table><tbody>
<tr>
<th align = 'left'> Directory/File </th>
<th align = 'left'> Description </th>
</tr>
<tr>
<td class='confluenceTd'> src/main/resources/config </td>
<td class='confluenceTd'> Contains monitor.xml and config.yml</td>
</tr>
<tr>
<td class='confluenceTd'> src/main/java </td>
<td class='confluenceTd'> Contains source code to the HanaDB monitoring extension </td>
</tr>
<tr>
<td class='confluenceTd'> target </td>
<td class='confluenceTd'> Only obtained when using maven. Run 'maven clean install' to get the distributable .zip file. </td>
</tr>
<tr>
<td class='confluenceTd'> pom.xml </td>
<td class='confluenceTd'> maven build script to package the project (required only if changing Java code) </td>
</tr>
</tbody>
</table>

## Metrics

Metrics can be configured in the `config.yml`file.

A reference for the [HanaDB System Views can be found here](https://help.sap.com/viewer/4fe29514fd584807ac9f2a04f6754767/2.0.00/en-US/20cbb10c75191014b47ba845bfe499fe.html)

The Queries can be adjusted to your needs to gather metrics from the HanaDB. Please see below example:

```
queries:
  - statement: "select * from M_DISK_USAGE where USED_SIZE >= 0"
    columns:
     - name: "HOST"
       type: "name"
     - name: "USAGE_TYPE"
       type: "name"
     - name: "USED_SIZE"
       type: "metric"
       convertFrom: ""
       convertTo: ""
  - statement: "select HOST, USED_PHYSICAL_MEMORY, FREE_PHYSICAL_MEMORY from M_HOST_RESOURCE_UTILIZATION"
    columns:
     - name: "HOST"
       type: "name"
     - name: "USED_PHYSICAL_MEMORY"
       type: "metric"
       convertFrom: ""
       convertTo: ""
     - name: "FREE_PHYSICAL_MEMORY"
       type: "metric"
       convertFrom: ""
       convertTo: ""

```

## Password Encryption Support

To avoid setting the clear text password in the `config.yml`. Please follow the process to encrypt the password and set the encrypted password and the key in the `config.yml`.

* Download the util jar to encrypt the password from [here](https://github.com/Appdynamics/maven-repo/raw/master/releases/com/appdynamics/appd-exts-commons/2.1.0/appd-exts-commons-2.1.0.jar)
* Encrypt password from the commandline
  * `java -cp appd-exts-commons-<VERSION>.jar com.appdynamics.extensions.crypto.Encryptor myKey myPassword`

## Contributing

Always feel free to fork and contribute any changes directly via [GitHub](https://github.com/michaelenglert/hanadb-monitoring-extension).

## Troubleshoot

1. Verify Machine Agent Data: Please start the Machine Agent without the extension and make sure that it reports data. Verify that the machine agent status is UP and it is reporting Hardware Metrics.
2. `config.yml`: Validate the file [here](http://www.yamllint.com/).
3. Metric Limit: Please start the machine agent with the argument `-Dappdynamics.agent.maxMetrics=5000` if there is a metric limit reached error in the logs. If you don't see the expected metrics, this could be the cause.
4. Check Logs: There could be some obvious errors in the machine agent logs. Please take a look.
5. `The config cannot be null` error: This usually happens when on a windows machine in `monitor.xml` you give `config.yml` file path with linux file path separator `/`. Use Windows file path separator `\` e.g. `monitors\Monitor\config.yml`. For Windows, please specify the complete path.
6. Collect Debug Logs: Edit the file, `<MachineAgent>/conf/logging/log4j.xml` and update the level of the appender `com.appdynamics` and `com.singularity` to debug. Let it run for 5-10 minutes and attach the logs to a support ticket.
7. If you encounter Exceptions with `[Read timed out]` please check the `communicationtimeout` setting within the `connectionString`. The query and the processing time might not fit into the specified window.

## Support

For any questions or feature request, please contact [AppDynamics Center of Excellence](mailto:help@appdynamics.com).
