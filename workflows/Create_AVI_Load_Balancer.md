# Create AVI Load Balancer - Workflow Documentation

<details>
<summary><h2>Workflow Details</h2></summary>

- **Workflow Name:** Create AVI Load Balancer
- **Workflow ID:** `e06ecd6e-df7d-470d-8249-3bb52f2845a6`
- **Description:** _No description provided_
</details>

<details>
<summary><h2>Workflow Variables</h2></summary>

| Name | Type |
|------|------|
| aviVersion | string |
| aviCreds | Properties |
| aviUrl | string |
| serverList | Array/string |
| health_payload | string |
| health_name | string |
| uuid | string |
| healthMonitor | string |
| poolPayload | Properties |
| pool | string |
| vipPayload | Properties |
| vsvip | string |
| vsPayload | Properties |
| aviTenant | Properties |
</details>

<details>
<summary><h2>Workflow Inputs</h2></summary>

| Name | Type |
|------|------|
| manualServers | Array/string |
| selectedVMs | Array/string |
| useVMSelection | boolean |
| useExistingHealthMonitor | boolean |
| existingHealthMonitor | string |
| health_monitor_type | string |
| healthPort | string |
| command_code | string |
| command_parameters | string |
| command_path | string |
| command_variables | string |
| lb_algorithm | string |
| poolPort | string |
| useExistingPool | boolean |
| existingPool | string |
| networkType | string |
| ipAddress | string |
| cloud | string |
| tier1_lr | string |
| serviceEngineGroup | string |
| vrfContext | string |
| network | string |
| networkSegment | string |
| vipIPRange | string |
| customerSuffix | string |
</details>

<details>
<summary><h2>Workflow Elements</h2></summary>

#### Element: item0
- **Display Name:** 
- **Type:** end
- **Description:** _No description provided_
---

#### Element: item1
- **Display Name:** initVariables
- **Type:** task
- **Description:** Simple task with custom script capability.
**Output Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| aviVersion | string | aviVersion |
| aviCreds | Properties | aviCreds |
| aviUrl | string | aviUrl |
| uuid | string | uuid |
| aviTenant | Properties | aviTenant |

**Script:**

```javascript
try {

    var automationModule = System.getModule("net.atos.dhc.automation");
    if (automationModule === null || automationModule === undefined) {
        throw new Error("Module 'net.atos.dhc.automation' is not available.");
    }
    
    var aviVersion = automationModule.getAVIVersion();
    if (aviVersion === null || aviVersion === undefined || (typeof aviVersion === "string" && aviVersion.trim() === "")) {
        throw new Error("aviVersion is required but not provided or is empty.");
    }
    
    var aviCreds = automationModule.getAviCreds();

    
    var aviUrl = automationModule.configElement("DHC", "SSRConfig", "aviServer");
    if (aviUrl === null || aviUrl === undefined || (typeof aviUrl === "string" && aviUrl.trim() === "")) {
        throw new Error("aviUrl is required but not provided or is empty.");
    }

    System.log(" [INFO] AVI Version: " + aviVersion + " / AVI URL: " + aviUrl);
    

    var aviTenant = automationModule.getAVITenantRef();
    System.log("AVI tenant reference: "+aviTenant.label +" with id: "+aviTenant.value)
    if (!aviTenant || !aviTenant.label) {
        throw new Error("AVI tenant reference or label is missing.");
    }

    var uuid = java.util.UUID.randomUUID().toString();
    timestamp = new Date().toISOString();
    System.log(" [INFO] Generated UUID: " + uuid);


    
} catch (e) {

    var timestamp = new Date().toISOString();
    System.log(" [ERROR] An error occurred during script execution: " + e.message);
    throw e;
}

```
---

#### Element: item2
- **Display Name:** mergeServers
- **Type:** task
- **Description:** Simple task with custom script capability.
**Input Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| useVMSelection | boolean | useVMSelection |
| manualServers | Array/string | manualServers |
| selectedVMs | Array/string | selectedVMs |

**Output Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| serverList | Array/string | serverList |

**Script:**

```javascript
try {
    var manualServers = manualServers || [];
    
    var selectedVMs = selectedVMs || [];

    var combined = manualServers.concat(useVMSelection ? selectedVMs : []);

    var deduped = [];
    for (var i = 0; i < combined.length; i++) {
        if (deduped.indexOf(combined[i]) === -1) {
            deduped.push(combined[i]);
        }
    }
    
    serverList = deduped;
    System.log(" [INFO] Merged Server List: " + JSON.stringify(serverList));
} catch (e) {
    System.log(" [ERROR] An error occurred during server list merging: " + e.message);
    throw e;
}

```
---

#### Element: item3
- **Display Name:** determineHealthMonitor
- **Type:** task
- **Description:** Simple task with custom script capability.
**Input Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| useExistingHealthMonitor | boolean | useExistingHealthMonitor |
| health_monitor_type | string | health_monitor_type |
| healthPort | string | healthPort |
| command_code | string | command_code |
| command_variables | string | command_variables |
| command_parameters | string | command_parameters |
| command_path | string | command_path |
| uuid | string | uuid |
| existingHealthMonitor | string | existingHealthMonitor |
| aviTenant | Properties | aviTenant |
| customerSuffix | string | customerSuffix |

**Output Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| health_payload | string | health_payload |
| health_name | string | health_name |
| healthMonitor | string | healthMonitor |

**Script:**

```javascript
try {
    if (useExistingHealthMonitor) {
        healthMonitor = existingHealthMonitor;
        System.log(" [INFO] Using existing health monitor: " + healthMonitor);
    } else {
        if (health_monitor_type == "HEALTH_MONITOR_EXTERNAL") {
            var health_payload = {
                "name": "external-monitor-" + uuid +customerSuffix,
                "type": health_monitor_type,
                "monitor_port": healthPort,
                "tenant_ref":aviTenant.value,
                "external_monitor": {
                    "command_code": command_code,
                    "command_parameters": command_parameters,
                    "command_path": command_path,
                    "command_variables": command_variables
                }
            };
            var body = JSON.stringify(health_payload);
            System.log(" [INFO] Creating External Health Monitor: " + body);
        } else {
            var health_payload = {
                "name": "standard-monitor-" + uuid + customerSuffix,
                "type": health_monitor_type,
                "monitor_port": healthPort,
                "monitor_port": healthPort
            };
            var body = JSON.stringify(health_payload);
            System.log(" [INFO] Creating Standard Health Monitor: " + body);
        }
        health_name = health_payload.name;
    }
} catch (e) {
    System.log(" [ERROR] An error occurred during health monitor creation: " + e.message);
    throw e;
}

```
---

#### Element: item4
- **Display Name:** createHealthMonitor
- **Type:** task
- **Description:** Simple task with custom script capability.
**Input Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| aviCreds | Properties | aviCreds |
| aviVersion | string | aviVersion |
| aviUrl | string | aviUrl |
| health_payload | string | health_payload |
| aviTenant | Properties | aviTenant |

**Output Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| healthMonitor | string | healthMonitor |

**Script:**

```javascript
try {
    var aviUsername = aviCreds.username;
    var aviPassword = aviCreds.password;

    var restHost = RESTHostManager.createTransientHostFrom(RESTHostManager.createHost("dynamicRequest"));
    restHost.name = "AVI";
    restHost.operationTimeout = 900;
    restHost.url = "https://" + aviUrl;

    var loginUri = "https://" + aviUrl + "/login";
    
    var ld = Config.getKeystores().getImportCAFromUrlAction();
    var model = ld.getModel();
    model.value = loginUri;
    var error = ld.execute();
    if (error) {
        System.error("Failed to import SSL certificate: " + error);
    } else {
        System.log("SSL certificate successfully imported.");
    }
    
    var loginPayload = JSON.stringify({ "username": aviUsername, "password": aviPassword });
    var loginRequest = restHost.createRequest("POST", loginUri, loginPayload);
    loginRequest.setHeader("Content-Type", "application/json");
    loginRequest.setHeader("Accept", "application/json");

    var loginResponse = loginRequest.execute();
    System.log("Login Status Code: " + loginResponse.statusCode);
    if (loginResponse.statusCode !== 200) {
        System.error("Login failed. Exiting.");
    }
    
    var cookies = loginResponse.getAllHeaders()['Set-Cookie'];
    var sessionId = cookies.match(/sessionid=([^;]+)/)[1];
    var csrfToken = cookies.match(/csrftoken=([^;]+)/)[1];

    System.log("Session ID: " + sessionId);
    System.log("CSRF Token: " + csrfToken);
    System.log("X-Avi-Version: " + aviVersion);

    var sessionCookie = "sessionid=" + sessionId;

    var body = JSON.stringify(health_payload);
    System.log("Prepared Payload: " + body);

    var apiUri = "https://" + aviUrl + "/api/healthmonitor";
    System.log("API URI: " + apiUri);

    var apiRequest = restHost.createRequest("POST", apiUri, body);
    apiRequest.setHeader("Content-Type", "application/json");
    apiRequest.setHeader("Accept", "application/json");
    apiRequest.setHeader("Cookie", sessionCookie);
    apiRequest.setHeader("X-CSRFToken", csrfToken);
    apiRequest.setHeader("X-Avi-Version", aviVersion);
    apiRequest.setHeader("Referer", "https://" + aviUrl + "/");
    apiRequest.setHeader("X-Avi-Tenant-UUID", aviTenant.value);
    apiRequest.setHeader("X-Avi-Tenant", aviTenant.label);

    var apiResponse = apiRequest.execute();
    System.log("API Request Status Code: " + apiResponse.statusCode);

    if (apiResponse.statusCode < 200 || apiResponse.statusCode >= 300) {
        throw new Error("API Request failed with status code " + apiResponse.statusCode + " and response "+ apiResponse.contentAsString);
    }

    System.log("API Response Content: " + apiResponse.contentAsString);

    var responseObj = JSON.parse(apiResponse.contentAsString);

    var uuid = responseObj.uuid;
    var monitorName = responseObj.name;

    healthMonitor = uuid + "#" + monitorName;
    
    System.log("Concatenated healthMonitor value: " + healthMonitor);
} catch (e) {
    System.log(" [ERROR] An error occurred during script execution: " + e.message);
    throw e;
}

```
---

#### Element: item6
- **Display Name:** useExistingHealthMonitor?
- **Type:** custom-condition
- **Description:** Custom decision based on a custom script.
**Input Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| useExistingHealthMonitor | boolean | useExistingHealthMonitor |

**Script:**

```javascript
try {
    if (useExistingHealthMonitor) {
        System.log(" [INFO] useExistingHealthMonitor is true.");
        return true;
    } else {
        System.log(" [INFO] useExistingHealthMonitor is false.");
        return false;
    }
} catch (e) {
    System.log(" [ERROR] An error occurred during health monitor selection: " + e.message);
    throw e;
}

```
---

#### Element: item7
- **Display Name:** determinePool
- **Type:** task
- **Description:** Simple task with custom script capability.
**Input Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| useExistingPool | boolean | useExistingPool |
| existingPool | string | existingPool |
| poolPort | string | poolPort |
| lb_algorithm | string | lb_algorithm |
| serverList | Array/string | serverList |
| healthMonitor | string | healthMonitor |
| uuid | string | uuid |
| aviUrl | string | aviUrl |
| cloud | string | cloud |
| tier1_lr | string | tier1_lr |
| networkSegment | string | networkSegment |
| aviTenant | Properties | aviTenant |
| customerSuffix | string | customerSuffix |

**Output Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| poolPayload | Properties | poolPayload |
| pool | string | pool |

**Script:**

```javascript
try {
    if (useExistingPool) {
        pool = existingPool;
        System.log(" [INFO] Using existing pool: " + pool);
    } else {
        var vrf_ref = networkSegment.split("@@")[2]
        var payload = {
            "name": "pool-" + uuid + customerSuffix,
            "default_server_port": poolPort,
            "lb_algorithm": lb_algorithm,
            "health_monitor_refs": [healthMonitor],
            "cloud_ref":cloud,
            "tier1_lr":tier1_lr,
            "vrf_ref": vrf_ref,
            "tenant_ref":aviTenant.value,
            "servers": []
        };

        for (var i = 0; i < serverList.length; i++) {
            payload.servers.push({
                "ip": { "addr": serverList[i], "type": "V4" }
            });
        }

        payload.health_monitor_refs = ["https://" + aviUrl + "/api/healthmonitor/" + healthMonitor];
        var body = JSON.stringify(payload);
        System.log(" [INFO] Creating Pool: " + body);
        poolPayload = payload;
    }
} catch (e) {
    System.log(" [ERROR] An error occurred during pool creation: " + e.message);
    throw e;
}

```
---

#### Element: item9
- **Display Name:** useExistingPool
- **Type:** custom-condition
- **Description:** Custom decision based on a custom script.
**Input Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| useExistingPool | boolean | useExistingPool |

**Script:**

```javascript
try {
    if (useExistingPool) {
        System.log(" [INFO] useExistingPool is true.");
        return true;
    } else {
        System.log(" [INFO] useExistingPool is false.");
        return false;
    }
} catch (e) {
    System.log(" [ERROR] An error occurred during pool selection: " + e.message);
    throw e;
}

```
---

#### Element: item10
- **Display Name:** createPool
- **Type:** task
- **Description:** Simple task with custom script capability.
**Input Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| aviUrl | string | aviUrl |
| aviCreds | Properties | aviCreds |
| aviVersion | string | aviVersion |
| poolPayload | Properties | poolPayload |
| aviTenant | Properties | aviTenant |

**Output Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| pool | string | pool |

**Script:**

```javascript
try {
    var aviUsername = aviCreds.username;
    var aviPassword = aviCreds.password;

    var restHost = RESTHostManager.createTransientHostFrom(RESTHostManager.createHost("dynamicRequest"));
    restHost.name = "AVI";
    restHost.operationTimeout = 900;
    restHost.url = "https://" + aviUrl;

    var loginUri = "https://" + aviUrl + "/login";
    var ld = Config.getKeystores().getImportCAFromUrlAction();
    var model = ld.getModel();
    model.value = loginUri;
    var error = ld.execute();
    if (error) {
        System.error("Failed to import SSL certificate: " + error);
    } else {
        System.log("SSL certificate successfully imported.");
    }

    var loginPayload = JSON.stringify({ "username": aviUsername, "password": aviPassword });
    var loginRequest = restHost.createRequest("POST", loginUri, loginPayload);
    loginRequest.setHeader("Content-Type", "application/json");
    loginRequest.setHeader("Accept", "application/json");

    var loginResponse = loginRequest.execute();
    System.log("Login Status Code: " + loginResponse.statusCode);
    if (loginResponse.statusCode !== 200) {
        System.error("Login failed. Exiting.");
    }

    var cookies = loginResponse.getAllHeaders()['Set-Cookie'];
    var sessionId = cookies.match(/sessionid=([^;]+)/)[1];
    var csrfToken = cookies.match(/csrftoken=([^;]+)/)[1];

    System.log("Session ID: " + sessionId);
    System.log("CSRF Token: " + csrfToken);
    System.log("X-Avi-Version: " + aviVersion);

    var sessionCookie = "sessionid=" + sessionId;

    var body = JSON.stringify(poolPayload);
    System.log("Prepared Payload: " + body);

    var apiUri = "https://" + aviUrl + "/api/pool";
    System.log("API URI: " + apiUri);

    var apiRequest = restHost.createRequest("POST", apiUri, body);
    apiRequest.setHeader("Content-Type", "application/json");
    apiRequest.setHeader("Accept", "application/json");
    apiRequest.setHeader("Cookie", sessionCookie);
    apiRequest.setHeader("X-CSRFToken", csrfToken);
    apiRequest.setHeader("X-Avi-Version", aviVersion);
    apiRequest.setHeader("Referer", "https://" + aviUrl + "/");
    apiRequest.setHeader("X-Avi-Tenant-UUID", aviTenant.value);
    apiRequest.setHeader("X-Avi-Tenant", aviTenant.label);
    
    var apiResponse = apiRequest.execute();
    System.log("API Request Status Code: " + apiResponse.statusCode);
    if (apiResponse.statusCode < 200 || apiResponse.statusCode >= 300) {
        throw new Error("API Request failed with status code " + apiResponse.statusCode + " and response "+ apiResponse.contentAsString);
    }
    System.log("API Response Content: " + apiResponse.contentAsString);

    var responseObj = JSON.parse(apiResponse.contentAsString);

    var uuid = responseObj.uuid;
    var poolName = responseObj.name;

    var pool = uuid + "#" + poolName;

    System.log("Concatenated pool value: " + pool);
} catch (e) {
    System.log(" [ERROR] An error occurred during pool creation: " + e.message);
    throw e;
}

```
---

#### Element: item11
- **Display Name:** determineVIP
- **Type:** task
- **Description:** Simple task with custom script capability.
**Input Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| networkType | string | networkType |
| ipAddress | string | ipAddress |
| uuid | string | uuid |
| cloud | string | cloud |
| tier1_lr | string | tier1_lr |
| vipIPRange | string | vipIPRange |
| networkSegment | string | networkSegment |
| aviTenant | Properties | aviTenant |
| customerSuffix | string | customerSuffix |

**Output Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| vipPayload | Properties | vipPayload |

**Script:**

```javascript
try {
    var vipPayload = {};
    var network_ref = networkSegment.split("@@")[1];
    var vrf_ref = networkSegment.split("@@")[2];
    if (networkType == "Static IP") {
        vipPayload = {
            "name": "vip-" + uuid + customerSuffix,
            "cloud_ref": cloud,
            "tier1_lr": tier1_lr,
            "tenant_ref":aviTenant.value,
            "vrf_ref": vrf_ref,
            "vip": [
                {
                    "ip_address": { "addr": ipAddress, "type": "V4" }
                }
            ]
        };
        var body = JSON.stringify(vipPayload);
        System.log(" [INFO] Creating Static VIP: " + body);
    } else {
        var addr = vipIPRange.split("/")[0];
        var mask = vipIPRange.split("/")[1];
        vipPayload = {
            "name": "vip-" + uuid + customerSuffix,
            "cloud_ref": cloud,
            "tier1_lr": tier1_lr,
            "vrf_ref": vrf_ref,
            "tenant_ref":aviTenant.value,
            "vip": [
                {
                    "auto_allocate_ip": true,
                    "ipam_network_subnet": {
                        "network_ref": network_ref,
                        "subnet": {
                            "ip_addr": {
                                "addr": addr,
                                "type": "V4"
                            },
                            "mask": mask
                        }
                    }
                }
            ]
        };
        var body = JSON.stringify(vipPayload);
        System.log(" [INFO] Creating Dynamic VIP: " + body);
    }
} catch (e) {
    System.log(" [ERROR] An error occurred during VIP creation: " + e.message);
    throw e;
}

```
---

#### Element: item14
- **Display Name:** createVSVIP
- **Type:** task
- **Description:** Simple task with custom script capability.
**Input Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| aviCreds | Properties | aviCreds |
| aviUrl | string | aviUrl |
| aviVersion | string | aviVersion |
| vipPayload | Properties | vipPayload |
| aviTenant | Properties | aviTenant |

**Output Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| vsvip | string | vsvip |

**Script:**

```javascript
try {
    var aviUsername = aviCreds.username;
    var aviPassword = aviCreds.password;

    var restHost = RESTHostManager.createTransientHostFrom(RESTHostManager.createHost("dynamicRequest"));
    restHost.name = "AVI";
    restHost.operationTimeout = 900;
    restHost.url = "https://" + aviUrl;

    var loginUri = "https://" + aviUrl + "/login";
    var ld = Config.getKeystores().getImportCAFromUrlAction();
    var model = ld.getModel();
    model.value = loginUri;
    var error = ld.execute();
    if (error) {
        System.error("Failed to import SSL certificate: " + error);
    } else {
        System.log("SSL certificate successfully imported.");
    }

    var loginPayload = JSON.stringify({ "username": aviUsername, "password": aviPassword });
    var loginRequest = restHost.createRequest("POST", loginUri, loginPayload);
    loginRequest.setHeader("Content-Type", "application/json");
    loginRequest.setHeader("Accept", "application/json");

    var loginResponse = loginRequest.execute();
    System.log("Login Status Code: " + loginResponse.statusCode);
    if (loginResponse.statusCode !== 200) {
        System.error("Login failed. Exiting.");
    }

    var cookies = loginResponse.getAllHeaders()['Set-Cookie'];
    var sessionId = cookies.match(/sessionid=([^;]+)/)[1];
    var csrfToken = cookies.match(/csrftoken=([^;]+)/)[1];

    System.log("Session ID: " + sessionId);
    System.log("CSRF Token: " + csrfToken);
    System.log("X-Avi-Version: " + aviVersion);

    var sessionCookie = "sessionid=" + sessionId;

    var body = JSON.stringify(vipPayload);
    System.log("Prepared Payload: " + body);

    var apiUri = "https://" + aviUrl + "/api/vsvip";
    System.log("API URI: " + apiUri);

    var apiRequest = restHost.createRequest("POST", apiUri, body);
    apiRequest.setHeader("Content-Type", "application/json");
    apiRequest.setHeader("Accept", "application/json");
    apiRequest.setHeader("Cookie", sessionCookie);
    apiRequest.setHeader("X-CSRFToken", csrfToken);
    apiRequest.setHeader("X-Avi-Version", aviVersion);
    apiRequest.setHeader("Referer", "https://" + aviUrl + "/");
    apiRequest.setHeader("X-Avi-Tenant-UUID", aviTenant.value);
    apiRequest.setHeader("X-Avi-Tenant", aviTenant.label);


    var apiResponse = apiRequest.execute();
    System.log("API Request Status Code: " + apiResponse.statusCode);
    if (apiResponse.statusCode < 200 || apiResponse.statusCode >= 300) {
        throw new Error("API Request failed with status code " + apiResponse.statusCode + " and response "+ apiResponse.contentAsString);
    }
    System.log("API Response Content: " + apiResponse.contentAsString);

    var responseObj = JSON.parse(apiResponse.contentAsString);

    var uuid = responseObj.uuid;

    var vipName = responseObj.name;

    var vsvip = uuid + "#" + vipName;

    System.log("Concatenated vsvip value: " + vsvip);
} catch (e) {
    System.log(" [ERROR] An error occurred during vsvip creation: " + e.message);
    throw e;
}

```
---

#### Element: item15
- **Display Name:** determineVirtualService
- **Type:** task
- **Description:** Simple task with custom script capability.
**Input Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| networkType | string | networkType |
| poolPort | string | poolPort |
| pool | string | pool |
| useExistingHealthMonitor | boolean | useExistingHealthMonitor |
| useExistingPool | boolean | useExistingPool |
| health_monitor_type | string | health_monitor_type |
| aviUrl | string | aviUrl |
| vsvip | string | vsvip |
| uuid | string | uuid |
| cloud | string | cloud |
| tier1_lr | string | tier1_lr |
| serviceEngineGroup | string | serviceEngineGroup |
| networkSegment | string | networkSegment |
| aviTenant | Properties | aviTenant |
| customerSuffix | string | customerSuffix |

**Output Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| vsPayload | Properties | vsPayload |

**Script:**

```javascript
try {
    var vsName = "";
    var vrf_ref = networkSegment.split("@@")[2];
    if (networkType == "Dynamic IP") {
        if (!useExistingPool && useExistingHealthMonitor) {
            vsName = "vs-dynamic-existing-hm-" + uuid + customerSuffix;
        } else if (!useExistingPool && !useExistingHealthMonitor && health_monitor_type == "HEALTH_MONITOR_EXTERNAL") {
            vsName = "vs-dynamic-new-hm-" + uuid + customerSuffix;
        } else if (useExistingPool) {
            vsName = "vs-dynamic-" + uuid + customerSuffix;
        } else if (!useExistingPool && !useExistingHealthMonitor && health_monitor_type != "HEALTH_MONITOR_EXTERNAL") {
            vsName = "vs-dynamic-new-hm-" + uuid + customerSuffix;
        }
    } else {
        if (!useExistingPool && !useExistingHealthMonitor && health_monitor_type != "HEALTH_MONITOR_EXTERNAL") {
            vsName = "vs-static-new-hm-" + uuid + customerSuffix;
        } else if (!useExistingPool && useExistingHealthMonitor) {
            vsName = "vs-static-existing-hm-" + uuid + customerSuffix;
        } else if (useExistingPool) {
            vsName = "vs-static-" + uuid + customerSuffix;
        } else if (!useExistingPool && !useExistingHealthMonitor && health_monitor_type == "HEALTH_MONITOR_EXTERNAL") {
            vsName = "vs-static-new-hm-" + uuid + customerSuffix;
        }
    }

    var vsPayload = {
        "name": vsName,
        "cloud_ref":cloud,
        "tier1_lr":tier1_lr,
        "vrf_ref": vrf_ref,
        "se_group_ref":serviceEngineGroup,
        "pool_ref": "https://" + aviUrl + "/api/pool/" + pool,
        "vsvip_ref": "https://" + aviUrl + "/api/vsvip/" + vsvip,
        "tenant_ref":aviTenant.value,
        "services": [{
            "port": poolPort,
            "enable_ssl": false
        }]
    };

    var body = JSON.stringify(vsPayload);
    System.log(" [INFO] Creating Virtual Service: " + body);
} catch (e) {
    System.log(" [ERROR] An error occurred during Virtual Service creation: " + e.message);
    throw e;
}

```
---

#### Element: item16
- **Display Name:** createVS
- **Type:** task
- **Description:** Simple task with custom script capability.
**Input Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| aviCreds | Properties | aviCreds |
| aviUrl | string | aviUrl |
| aviVersion | string | aviVersion |
| vsPayload | Properties | vsPayload |
| aviTenant | Properties | aviTenant |

**Script:**

```javascript
try {
    var aviUsername = aviCreds.username;
    var aviPassword = aviCreds.password;

    var restHost = RESTHostManager.createTransientHostFrom(RESTHostManager.createHost("dynamicRequest"));
    restHost.name = "AVI";
    restHost.operationTimeout = 900;
    restHost.url = "https://" + aviUrl;

    var loginUri = "https://" + aviUrl + "/login";
    var ld = Config.getKeystores().getImportCAFromUrlAction();
    var model = ld.getModel();
    model.value = loginUri;
    var error = ld.execute();
    if (error) {
        System.error("Failed to import SSL certificate: " + error);
    } else {
        System.log("SSL certificate successfully imported.");
    }

    var loginPayload = JSON.stringify({ "username": aviUsername, "password": aviPassword });
    var loginRequest = restHost.createRequest("POST", loginUri, loginPayload);
    loginRequest.setHeader("Content-Type", "application/json");
    loginRequest.setHeader("Accept", "application/json");

    var loginResponse = loginRequest.execute();
    System.log("Login Status Code: " + loginResponse.statusCode);
    if (loginResponse.statusCode !== 200) {
        System.error("Login failed. Exiting.");
    }

    var cookies = loginResponse.getAllHeaders()['Set-Cookie'];
    var sessionId = cookies.match(/sessionid=([^;]+)/)[1];
    var csrfToken = cookies.match(/csrftoken=([^;]+)/)[1];

    System.log("Session ID: " + sessionId);
    System.log("CSRF Token: " + csrfToken);
    System.log("X-Avi-Version: " + aviVersion);

    var sessionCookie = "sessionid=" + sessionId;

    var body = JSON.stringify(vsPayload);
    System.log("Prepared Payload: " + body);

    var apiUri = "https://" + aviUrl + "/api/virtualservice";
    System.log("API URI: " + apiUri);

    var apiRequest = restHost.createRequest("POST", apiUri, body);
    apiRequest.setHeader("Content-Type", "application/json");
    apiRequest.setHeader("Accept", "application/json");
    apiRequest.setHeader("Cookie", sessionCookie);
    apiRequest.setHeader("X-CSRFToken", csrfToken);
    apiRequest.setHeader("X-Avi-Version", aviVersion);
    apiRequest.setHeader("Referer", "https://" + aviUrl + "/");
    apiRequest.setHeader("X-Avi-Tenant-UUID", aviTenant.value);
    apiRequest.setHeader("X-Avi-Tenant", aviTenant.label);

    var apiResponse = apiRequest.execute();
    System.log("API Request Status Code: " + apiResponse.statusCode);
    if (apiResponse.statusCode < 200 || apiResponse.statusCode >= 300) {
        throw new Error("API Request failed with status code " + apiResponse.statusCode + " and response "+ apiResponse.contentAsString);
    }
    System.log("API Response Content: " + apiResponse.contentAsString);

    var responseObj = JSON.parse(apiResponse.contentAsString);

    var uuid = responseObj.uuid;

    var vsName = responseObj.name;

    var virtualservice = uuid + "#" + vsName;

    System.log("Concatenated virtualservice value: " + virtualservice);
} catch (e) {
    System.log(" [ERROR] An error occurred during virtualservice creation: " + e.message);
    throw e;
}

```
---

#### Element: item17
- **Display Name:** End workflow
- **Type:** end
- **Description:** _No description provided_
---

#### Element: item18
- **Display Name:** 
- **Type:** end
- **Description:** _No description provided_
---

#### Element: item19
- **Display Name:** Check Health Monitor
- **Type:** custom-condition
- **Description:** Custom decision based on a custom script.
**Input Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| healthMonitor | string | healthMonitor |
| useExistingHealthMonitor | boolean | useExistingHealthMonitor |

**Script:**

```javascript
if(healthMonitor === "" || useExistingHealthMonitor === true)
{
    return false
}
else
{
    return true
}
```
---

#### Element: item20
- **Display Name:** clear Health Monitor
- **Type:** task
- **Description:** Simple task with custom script capability.
**Input Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| aviCreds | Properties | aviCreds |
| aviUrl | string | aviUrl |
| aviVersion | string | aviVersion |
| healthMonitor | string | healthMonitor |

**Script:**

```javascript
try {
    var aviUsername = aviCreds.username;
    var aviPassword = aviCreds.password;

    var restHost = RESTHostManager.createTransientHostFrom(RESTHostManager.createHost("dynamicRequest"));
    restHost.name = "AVI";
    restHost.operationTimeout = 900;
    restHost.url = "https://" + aviUrl;

    var loginUri = "https://" + aviUrl + "/login";
    
    var ld = Config.getKeystores().getImportCAFromUrlAction();
    var model = ld.getModel();
    model.value = loginUri;
    var error = ld.execute();
    if (error) {
        System.error("Failed to import SSL certificate: " + error);
    } else {
        System.log("SSL certificate successfully imported.");
    }
    
    var loginPayload = JSON.stringify({ "username": aviUsername, "password": aviPassword });
    var loginRequest = restHost.createRequest("POST", loginUri, loginPayload);
    loginRequest.setHeader("Content-Type", "application/json");
    loginRequest.setHeader("Accept", "application/json");

    var loginResponse = loginRequest.execute();
    System.log("Login Status Code: " + loginResponse.statusCode);
    if (loginResponse.statusCode !== 200) {
        System.error("Login failed. Exiting.");
    }
    
    var cookies = loginResponse.getAllHeaders()['Set-Cookie'];
    var sessionId = cookies.match(/sessionid=([^;]+)/)[1];
    var csrfToken = cookies.match(/csrftoken=([^;]+)/)[1];

    System.log("Session ID: " + sessionId);
    System.log("CSRF Token: " + csrfToken);
    System.log("X-Avi-Version: " + aviVersion);

    var sessionCookie = "sessionid=" + sessionId;


    var apiUri = "https://" + aviUrl + "/api/healthmonitor/"+healthMonitor;
    System.log("API URI: " + apiUri);

    var apiRequest = restHost.createRequest("DELETE", apiUri);
    apiRequest.setHeader("Content-Type", "application/json");
    apiRequest.setHeader("Accept", "application/json");
    apiRequest.setHeader("Cookie", sessionCookie);
    apiRequest.setHeader("X-CSRFToken", csrfToken);
    apiRequest.setHeader("X-Avi-Version", aviVersion);
    apiRequest.setHeader("Referer", "https://" + aviUrl + "/");

    var apiResponse = apiRequest.execute();
    System.log("API Request Status Code: " + apiResponse.statusCode);

    if (apiResponse.statusCode < 200 || apiResponse.statusCode >= 300) {
        throw new Error("API Request failed with status code " + apiResponse.statusCode + " and response "+ apiResponse.contentAsString);
    }

    System.log("API Response Content: " + apiResponse.contentAsString);




} catch (e) {
    System.log(" [ERROR] An error occurred during script execution: " + e.message);
    throw e;
}

```
---

#### Element: item22
- **Display Name:** Check Pool
- **Type:** custom-condition
- **Description:** Custom decision based on a custom script.
**Input Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| pool | string | pool |
| useExistingPool | boolean | useExistingPool |

**Script:**

```javascript
if(pool === "" || useExistingPool === true)
{
    return false
}
else
{
    return true
}
```
---

#### Element: item23
- **Display Name:** Clear Pool
- **Type:** task
- **Description:** Simple task with custom script capability.
**Input Bindings:**

| Variable Name | Type | Workflow Variable |
|---------------|------|-------------------|
| aviCreds | Properties | aviCreds |
| aviUrl | string | aviUrl |
| aviVersion | string | aviVersion |
| pool | string | pool |

**Script:**

```javascript
try {
    var aviUsername = aviCreds.username;
    var aviPassword = aviCreds.password;

    var restHost = RESTHostManager.createTransientHostFrom(RESTHostManager.createHost("dynamicRequest"));
    restHost.name = "AVI";
    restHost.operationTimeout = 900;
    restHost.url = "https://" + aviUrl;

    var loginUri = "https://" + aviUrl + "/login";
    
    var ld = Config.getKeystores().getImportCAFromUrlAction();
    var model = ld.getModel();
    model.value = loginUri;
    var error = ld.execute();
    if (error) {
        System.error("Failed to import SSL certificate: " + error);
    } else {
        System.log("SSL certificate successfully imported.");
    }
    
    var loginPayload = JSON.stringify({ "username": aviUsername, "password": aviPassword });
    var loginRequest = restHost.createRequest("POST", loginUri, loginPayload);
    loginRequest.setHeader("Content-Type", "application/json");
    loginRequest.setHeader("Accept", "application/json");

    var loginResponse = loginRequest.execute();
    System.log("Login Status Code: " + loginResponse.statusCode);
    if (loginResponse.statusCode !== 200) {
        System.error("Login failed. Exiting.");
    }
    
    var cookies = loginResponse.getAllHeaders()['Set-Cookie'];
    var sessionId = cookies.match(/sessionid=([^;]+)/)[1];
    var csrfToken = cookies.match(/csrftoken=([^;]+)/)[1];

    System.log("Session ID: " + sessionId);
    System.log("CSRF Token: " + csrfToken);
    System.log("X-Avi-Version: " + aviVersion);

    var sessionCookie = "sessionid=" + sessionId;


    var apiUri = "https://" + aviUrl + "/api/pool/"+pool;
    System.log("API URI: " + apiUri);

    var apiRequest = restHost.createRequest("DELETE", apiUri);
    apiRequest.setHeader("Content-Type", "application/json");
    apiRequest.setHeader("Accept", "application/json");
    apiRequest.setHeader("Cookie", sessionCookie);
    apiRequest.setHeader("X-CSRFToken", csrfToken);
    apiRequest.setHeader("X-Avi-Version", aviVersion);
    apiRequest.setHeader("Referer", "https://" + aviUrl + "/");

    var apiResponse = apiRequest.execute();
    System.log("API Request Status Code: " + apiResponse.statusCode);

    if (apiResponse.statusCode < 200 || apiResponse.statusCode >= 300) {
        throw new Error("API Request failed with status code " + apiResponse.statusCode + " and response "+ apiResponse.contentAsString);
    }

    System.log("API Response Content: " + apiResponse.contentAsString);

  


} catch (e) {
    System.log(" [ERROR] An error occurred during script execution: " + e.message);
    throw e;
}

```
---

</details>

<details>
<summary><h2>Error Handlers</h2></summary>

- **Element Name:** item22
- **Throws Variable:** _None_
</details>

<details>
<summary><h2>Linked Workflows</h2></summary>

_No linked workflows defined._
</details>

<details>
<summary><h2>Linked Actions Documentation</h2></summary>

# Actions Documentation


</details>

