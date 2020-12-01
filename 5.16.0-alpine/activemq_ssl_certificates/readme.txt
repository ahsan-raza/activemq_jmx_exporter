this readme containes files which are to be used with activemq ssl configuration Stomp & OpenWire
every certificate usage is described below:
=============================================== ACTIVEMQ CONFIGURATION ===============================================
NOTE: Sample activemq.xml file is also part of this archive
1. broker.ks
	copy this file to {activemq.base}/conf/ directory.
2. broker.ts
	copy this file to {activemq.base}/conf/ directory.
	Open {activemq.base}/conf/activemq.xml file with any IDE and add sslContext. Add broker.ks and broker.ts file names
	in it. Password for both certificates is "password"
	Example:
	sslContext:
		<sslContext> 
			<sslContext keyStore="file:${activemq.base}/conf/broker.ks"
			  keyStorePassword="password" trustStore="file:${activemq.base}/conf/broker.ts"
			  trustStorePassword="password"/> 
	    </sslContext> 
	add tranportConnectors e.g.
		<transportConnectors>
            <!-- DOS protection, limit concurrent connections to 1000 and frame size to 100MB -->
            <transportConnector name="openwire" uri="tcp://0.0.0.0:61616?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="amqp" uri="amqp://0.0.0.0:5672?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="stomp" uri="stomp://0.0.0.0:61613?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="mqtt" uri="mqtt://0.0.0.0:1883?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="ws" uri="ws://0.0.0.0:61614?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="ssl" uri="ssl://0.0.0.0:61617?transport.enabledProtocols=TLSv1.2"/>
            <transportConnector name="stomp+ssl" uri="stomp+nio+ssl://0.0.0.0:61615?transport.enabledProtocols=TLSv1.2&amp;needClientAuth=true"/>
        </transportConnectors>

================================ COMMUNICATION SERVER AND ROUTING ENGINE CONFIGURATION ==============================
NOTE: Set SSL=true in config.properties if you want to use SSL transport with ActiveMQ. use port 61617 if SSL is true, else use port 61616

3. client.ks
	client keyStore is needed in CommunicationServer and RoutingEngine projects, copy this file to any directory and
	enter absolute path of these files in CommunicationServer => config.properties & RoutingEngine => config.properties
	e.g.
		KEYSTORE_PATH=F:\\certs\\client.ks
	Password of this keyStore is "password". Add password in both config.properties i.e.:
		KEYSTORE_PWD=password
4. client.ts
	client trustStore is needed in CommunicationServer and RoutingEngine projects, copy this file to any directory and
	enter absolute path of these files in CommunicationServer => config.properties & RoutingEngine => config.properties
	e.g.
		TRUSTSTORE_PATH=F:\\certs\\client.ts
	Password of this trustStore is "password". Add password in both config.properties i.e.:
		TRUSTSTORE_PWD=password

============================================= CHAT SERVER CONFIGURATION ===============================================
NOTE: Set tls to true in config.ActiveMQ.tls if you want to use ssl+stomp client. For Stomp client, use port 61613, for Stomp+ssl client, use port 61615

5. broker.pem
	This is cacerts authority for Node.js. Copy this file to any directory and put absolute path of broker.pem in "certificateAuthorityPath" section of following Chat Server config:
		"ActiveMQ": {
			"TaskDestination": "Task",
			"GeneralQueue": "General",
			"broker": {
			  "host": "localhost",
			  "port": 61615,
			  "user": "",
			  "pass": "",
			  "reconnectOpts": {
				"retries": 20,
				"delay": 500
			  }
			},
			"tls": "true",
			.....,
			......,
			"certificateAuthorityPath": "F:\\certs\\broker.pem",
			"passphrase": "password"  
		  }
	Password of broker.pem is "password"
6. client.key
	client key file for Chat Server. Copy this file to any directory and put absolute path into "certificateKeyPath" section of following configuration:
		"ActiveMQ": {
			"TaskDestination": "Task",
			"GeneralQueue": "General",
			"broker": {
			  "host": "localhost",
			  "port": 61615,
			  "user": "",
			  "pass": "",
			  "reconnectOpts": {
				"retries": 20,
				"delay": 500
			  }
			},
			"tls": "true",
			......,
			"certificateKeyPath": "F:\\certs\\client.key",
			.......,
			.......  
		  }
7. client.pem
	client certificate file for Chat Server. Copy this file to any directory and put absolute path into "certificatePath" section of following configuration:
		"ActiveMQ": {
			"TaskDestination": "Task",
			"GeneralQueue": "General",
			"broker": {
			  "host": "localhost",
			  "port": 61615,
			  "user": "",
			  "pass": "",
			  "reconnectOpts": {
				"retries": 20,
				"delay": 500
			  }
			},
			"tls": "true",
			"certificatePath": "F:\\certs\\client.pem",
			........,
			........,
			........
		  }
=============================================== THAT'S ALL FOLKS =================================================