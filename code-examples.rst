Code examples
=============

Below are some code example for basic SentinelDB functionality:

Inserting a single entry
***********************

.. content-tabs::

	.. tab-container:: java
		:title: Java
		
		The Java example uses the `sentineldb-java-client <https://github.com/LogSentinel/logsentinel-java-client/>`_ 
		
		.. code-block:: java
		
			//credentials obtained after registration
			LogSentinelClientBuilder builder = LogSentinelClientBuilder
			    .create(applicationId, organizationId, secret);
			LogSentinelClient client = builder.build();

			try {
			    LogResponse result = client.getAuditLogActions().log(
				new ActorData(actorId).setActorDisplayName(username).setActorRoles(roles), 
				new ActionData(details).setAction(action)
			    );
			    System.out.println(result);
			} catch (ApiException e) {
			    System.err.println("Exception when calling AuditLogControllerApi#logAuthAction");
			    e.printStackTrace();
			}
			
	.. tab-container:: java
		:title: C#
		
		The C# example uses the `logsentinel-dotnet-core-client <https://github.com/LogSentinel/logsentinel-dotnet-core-client/>`_ 
		
		.. code-block:: java
		
			Console.WriteLine(result.LogEntryId);
			
	.. tab-container:: php
		:title: PHP
		
		.. code-block:: php
		
			$data = <<<EOT
			{
			  "detail1": "detail 1",
			  "detail2": "detail 2"
			}
			EOT;
			
			$curl = curl_init();
			curl_setopt($curl, CURLOPT_POST, 1);
			curl_setopt($curl, CURLOPT_POSTFIELDS, $data);
			
			curl_setopt($curl, CURLOPT_URL, 'https://app.logsentinel.com/api/log/' + actorId + '/' + action + '/' + entityType + '/' + entityId);
			curl_setopt($curl, CURLOPT_HTTPHEADER, array(
				'Content-Type: application/json'
			));
			curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
			curl_setopt($curl, CURLOPT_HTTPAUTH, CURLAUTH_BASIC);
			curl_setopt($curl, CURLOPT_USERPWD, $ORG_ID . ":" . $SECRET);
			
			// EXECUTE:
			$result = curl_exec($curl);	
			
	.. tab-container:: python
		:title: Python
		
		.. code-block:: python
			
			import requests
			url = 'https://app.logsentinel.com/api/log/' + actorId + '/' + action + '/' + entityType + '/' + entityId);
			data = '''{
			  "detail1": "detail 1",
			  "detail2": "detail 2"
			}'''
			
			response = requests.post(url, auth=HTTPBasicAuth(orgId, secret), data=data, headers={"Content-Type": "application/json"})
		
	.. tab-container:: nodejs
		:title: Node.js

		.. code-block:: javascript
		
			var https = require('https');
			var data = JSON.stringify({
			  "detail1": "detail 1",
			  "detail2": "detail 2"
			});

			var auth = 'Basic ' + Buffer.from(ORG_ID + ':' + ORG_SECRET).toString('base64')

			var options = {
			  host: 'app.logsentinel.com',
			  path: '/api/log/' + actorId + '/' + action + '/' + entityType + '/' + entityId,
			  method: 'POST',
			  headers: {
				'Content-Type': 'application/json; charset=utf-8',
				'Content-Length': data.length
				'Authorization': auth;
			  }
			};

			var req = https.request(options, function(res) {
			  var res = JSON.parse(response.body)
			  //...
			});

			req.write(data);
			req.end();
			
Inserting batch entries
***********************

.. content-tabs::

	.. tab-container:: java
		:title: Java
		
		The Java example uses the `sentineldb-java-client <https://github.com/LogSentinel/logsentinel-java-client/>`_ 
		
		.. code-block:: java
		
			//credentials obtained after registration
			LogSentinelClientBuilder builder = LogSentinelClientBuilder
			    .create(applicationId, organizationId, secret);
			LogSentinelClient client = builder.build();
			
			List<BatchLogRequestEntry> batch = new ArrayList<>();
			for (int i = 0; i < COUNT; i++) {
			    String details = "detais" + i;

			    BatchLogRequestEntry entry = new BatchLogRequestEntry();
			    entry.setActionData(new ActionData(details).setAction(action).setBinaryContent(false) );
			    entry.setActorData(new ActorData(actorId).setActorDisplayName(username).setActorRoles(roles).setDepartment("IT"));
			    entry.setAdditionalParams(new HashMap<>());

			    batch.add(entry);
			}

			try {
			    client.getAuditLogActions().logBatch(batch);
			} catch (ApiException e) {
			    // handle exception
			}
			
	.. tab-container:: C#
		:title: C#
		
		The C# example uses the `logsentinel-dotnet-core-client <https://github.com/LogSentinel/logsentinel-dotnet-core-client/>`_ 
		
		.. code-block:: C#
		
			
			Console.WriteLine(result.LogEntryId);
			
			
	.. tab-container:: php
		:title: PHP
		
		.. code-block:: php
		
			$data = <<<EOT
			[{
			  "actorData": {
			    "actorId":"actor1",
			    "actorDisplayName":"actor 1",
			    "department":"IT"
			  },
			  "actionData": {
			    "action":"VIEW",
			    "entityId":"123",
			    "entityType":"Deposit",
			    "details":{
				  "detail1": "detail 1",
				  "detail2": "detail 2"
			    }
			},{
			  "actorData": {
			    "actorId":"actor2",
			    "actorDisplayName":"actor 2",
			    "department":"IT"
			  },
			  "actionData": {
			    "action":"WITHDRAW",
			    "entityId":"123",
			    "entityType":"Deposit",
			    "details":{
				  "detail1": "detail 1",
				  "detail2": "detail 2"
			    }]
			EOT;
			
			$curl = curl_init();
			curl_setopt($curl, CURLOPT_POST, 1);
			curl_setopt($curl, CURLOPT_POSTFIELDS, $data);
			
			curl_setopt($curl, CURLOPT_URL, 'https://app.logsentinel.com/api/log/batch');
			curl_setopt($curl, CURLOPT_HTTPHEADER, array(
				'Content-Type: application/json'
			));
			curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
			curl_setopt($curl, CURLOPT_HTTPAUTH, CURLAUTH_BASIC);
			curl_setopt($curl, CURLOPT_USERPWD, $ORG_ID . ":" . $SECRET);
			
			// EXECUTE:
			$result = curl_exec($curl);	
			
	.. tab-container:: python
		:title: Python
		
		.. code-block:: python
			
			import requests
			url = 'https://app.logsentinel.com/api/log/batch');
			data = '''[{
			  "actorData": {
			    "actorId":"actor1",
			    "actorDisplayName":"actor 1",
			    "department":"IT"
			  },
			  "actionData": {
			    "action":"VIEW",
			    "entityId":"123",
			    "entityType":"Deposit",
			    "details":{
				  "detail1": "detail 1",
				  "detail2": "detail 2"
			    }
			},{
			  "actorData": {
			    "actorId":"actor2",
			    "actorDisplayName":"actor 2",
			    "department":"IT"
			  },
			  "actionData": {
			    "action":"WITHDRAW",
			    "entityId":"123",
			    "entityType":"Deposit",
			    "details":{
				  "detail1": "detail 1",
				  "detail2": "detail 2"
			    }]'''
			
			response = requests.post(url, auth=HTTPBasicAuth(orgId, secret), data=data, headers={"Content-Type": "application/json"})
			
	.. tab-container:: nodejs
		:title: Node.js

		.. code-block:: javascript
		
			var https = require('https');
			var data = JSON.stringify([{
			  "actorData": {
			    "actorId":"actor1",
			    "actorDisplayName":"actor 1",
			    "department":"IT"
			  },
			  "actionData": {
			    "action":"VIEW",
			    "entityId":"123",
			    "entityType":"Deposit",
			    "details":{
				  "detail1": "detail 1",
				  "detail2": "detail 2"
			    }
			},{
			  "actorData": {
			    "actorId":"actor2",
			    "actorDisplayName":"actor 2",
			    "department":"IT"
			  },
			  "actionData": {
			    "action":"WITHDRAW",
			    "entityId":"123",
			    "entityType":"Deposit",
			    "details":{
				  "detail1": "detail 1",
				  "detail2": "detail 2"
			    }]);

			var auth = 'Basic ' + Buffer.from(ORG_ID + ':' + ORG_SECRET).toString('base64')

			var options = {
			  host: 'app.logsentinel.com',
			  path: '/api/log/batch',
			  method: 'POST',
			  headers: {
				'Content-Type': 'application/json; charset=utf-8',
				'Content-Length': data.length
				'Authorization': auth;
			  }
			};

			var req = https.request(options, function(res) {
			  var res = JSON.parse(response.body)
			  //...
			});

			req.write(data);
			req.end();
			
			
