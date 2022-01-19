# BUILD of 2 distributed, scalable Vue JS single page web apps, based on vert-x rx-java toolkit:

        1. dashboard-webapp-vert-x                      # list IoT devices, location, traffic data throughput, rank
        2. user-webapp-vert-x                           # admin of the device, and his/her coordindates e.g. email


        cd dwebapp-vert-x <enter>
        mvn clean install -D skipTests <enter>          # build.log file
        
        # OUT: 
        scalable-vue-webapps-vert-x-1.0-all.jar 
        refers two distributed, scalable web apps: 
        dashboard-webapp-vert-x-1.0-all.jar, 
        user-webapp-vert-x-1.0.-all.jar 

# Prerequisite sytem services: Postgres, Mongo, Zookeeper, Kafka

        ```bash
        $ docker-compose up <enter>
        # OUT:
                Creating network "dwebapp-vert-x_default" with the default driver
                Creating dwebapp-vert-x_mailhog_1   ... done
                Creating dwebapp-vert-x_zookeeper_1 ... done
                Creating dwebapp-vert-x_artemis_1   ... done
                Creating dwebapp-vert-x_mongo_1     ... done
                Creating dwebapp-vert-x_postgres_1  ... done
                Creating dwebapp-vert-x_kafka_1     ... done
                ...
        ```

        # OUT: (docker image instances running)
        dwebapp-vert-x_kafka_1, strimzi/kafka, port 9092
        dwebapp-vert-x_mongo_1, mongo4:/bionic, port 27017
        dwebapp-vert-x_postrgress_1 postres:11-alpina, port 5432
        dwebapp-vert-x_aretmis_1 activemq posrt 5672
        dwebapp-vert-x_zookeeper_1 strimzi/zookeeper, port 2181

# install hivemind

        ```bash
        ntrajic@DESKTOP-6PK7L32:/mnt/c/SRC/Java/dwebapp-vert-x$ go get -u -f github.com/DarthSim/hivemind
                go get: -f flag is a no-op when using modules
                go: downloading github.com/DarthSim/hivemind v1.1.0
                go: downloading golang.org/x/sys v0.0.0-20220114195835-da31bd327af9
                go get: installing executables with 'go get' in module mode is deprecated.
                        Use 'go install pkg@version' instead.
                        For more information, see https://golang.org/doc/go-get-install-deprecation
                        or run 'go help get' or 'go help install'.
        ```

# Running Vert-x webapp services: spawning microservices w/ hivemind



        ```bash
        $ hivemind <enter>                              # will spawn services listed in Proc file
        ```

        Proc file content:
        user-webapp-vert-x: java -jar user-webapp-vert-x/target/user-webapp-vert-x-1.0-all.jar
        dashboard-webapp-vert-x: java -jar dashboard-webapp-vert-x/target/dashboard-webapp-vert-x-1.0-all.jar

        ```
        ntrajic@DESKTOP-6PK7L32:/mnt/c/SRC/Java/dwebapp-vert-x$ hivemind
        # OUT:
                user-webapp-vert-x      | Running...
                dashboard-webapp-vert-x | Running...
                
                dashboard-webapp-vert-x | 2022-01-18 15:45:42,257 INFO [vert.x-eventloop-thread-0] org.apache.kafka.clients.consumer.ConsumerConfig - ConsumerConfig values: 
                dashboard-webapp-vert-x |       allow.auto.create.topics = true
                kafka.client.serialization.JsonObjectDeserializer
                dashboard-webapp-vert-x |
                dashboard-webapp-vert-x | 2022-01-18 15:45:42,539 INFO [vert.x-eventloop-thread-0] org.apache.kafka.common.utils.AppInfoParser - Kafka version: 2.6.0
                dashboard-webapp-vert-x | 2022-01-18 15:45:42,540 INFO [vert.x-eventloop-thread-0] org.apache.kafka.common.utils.AppInfoParser - Kafka commitId: 62abe01bee039651
                dashboard-webapp-vert-x | 2022-01-18 15:45:42,540 INFO [vert.x-eventloop-thread-0] org.apache.kafka.common.utils.AppInfoParser - Kafka startTimeMs: 1642549542539
        ```

# Vue webapp UI

        1. The Vert.x web module makes it easy to build an edge service with CORS support and HTTP calls to other services.

        2. JSON web tokens are useful for authorization and access control in a public API.

        3. Vert.x does not have a preference regarding frontend application frameworks, but it is easy to integrate a Vue.js frontend application.

        4. By combining Docker containers managed from Testcontainers and the Rest Assured library, you can write integration tests for HTTP APIs.

        https://vertx.io/docs/vertx-web/java/#_using_vert_x_web
        https://github.com/vert-x3/vertx-web
        https://gitee.com/q651231292/Vertx-Web-Demo#vertx-web-examples

        The following Vert.x modules are needed to implement the public API:
        1. vertx-web, to provide advanced HTTP request-processing functionality
        2. vertx-web-client, to issue HTTP requests to the user profile service
        3. vertx-auth-jwt, to generate and process JSON web tokens and perform access control

        Client ----> Router ----> JWT Handler ---> CheckDevice  ---> CheckTrafficVolume
                              access ctrl      dev details
                                o--> JWT token
        Getting JWT token:
        ```bash
        $ http :4000/api/v1/token username=foo password=123     # --> a JWT token: header.payload.signature
        ```                                                     # hdr: {'typ': 'JWT', 'algo': 'RS256'}
                                                                # payload: { 'dev_id': '12a3', 'sub': 'foo', 'iss': '11123', 'exp': '2325' }
                                                                # sub = subject, iss = issurer of token, exp = token expiration
        The vertx-web module provides a router that can act as a Vert.x HTTP server request handler, and that manages the dispatch of HTTP requests to suitable handlers based on request paths (e.g., /foo) and HTTP methods (e.g., POST).


# Vue JavaScript web UI: Front End

        file: AppVue.js structure:

                <template>                                              // HTML template
                  <div id="app">
                    {{ hello }}                                         // property var           
                  </div>
                </template>

                <style scoped>                                          // local CSS                  
                  div {
                    border: solid 1px black;
                  }
                </style>

                <script>
                  export default {
                    data() {
                      return {
                        hello: "Hello, world!"                          // property var initialized
                }
                }
                }
                </script>

# Vue JS - FE JavaScript App structure:

        src/main.js--The entry point
        src/router.js--The Vue.js router that dispatches to the components of the three different screens
        src/DataStore.js--An object to hold the application store using the web browser local storage API, shared among all screens
        src/App.vue--The main component that mounts the Vue.js router
        src/views--Contains the three screen components: Home.vue, Login.vue, and Register.vue

# Vue JavaScript and Vert.x Bridge: 

        https://github.com/vert-x3/vertx-eventbus-bridge-clients 
        https://github.com/vert-x3/vertx-eventbus-bridge-clients/tree/master/nodejs
        https://github.com/vert-x3/vertx-eventbus-bridge-clients/tree/master/javascript

        package.json
                "dependencies": {
                        "core-js": "^2.6.5",
                        "moment": "^2.24.0",                            # formatting timestamps, dates
                        "vertx3-eventbus-client": "^3.7.1",             # vert.x <--> javascript integration
                        "vue": "^2.6.10"                                # front end single page framework
                },

        App.vue
                <script>
                import EventBus from 'vertx3-eventbus-client'   // <------ vertx3-eventbus-client
                import moment from 'moment'

                const eventBus = new EventBus("/eventbus")
                eventBus.enableReconnect(true)

                export default {
                data() {
                return {
                        throughput: 0,
                        cityTrendData: {},
                        publicRanking: []
                }
                },
                mounted() {
                eventBus.onopen = () => {
                        eventBus.registerHandler("client.updates.throughput", (err, message) => {
                        this.throughput = message.body.throughput
                        })
                        eventBus.registerHandler("client.updates.city-trend", (err, message) => {
                        const data = message.body
                        data.moment = moment(data.timestamp)
                        this.$set(this.cityTrendData, message.body.city, data)
                        })
                        eventBus.registerHandler("client.updates.publicRanking", (err, message) => {
                        this.publicRanking = message.body
                        })
                }
                },
                computed: {
                cityTrendRanking: function () {
                        const values = Object.values(this.cityTrendData).slice(0)
                        values.sort((a, b) => b.stepsCount - a.stepsCount)
                        return values
                }
                },
                }
                </script>

# com.moowork.node maven plugin for Node JS

        Login and registration of IoT dev admins


# Vue UI Single Pages

## user-webapp-vert-x UI

        http://localhost:8080/#/login

        User name
                INPUT...somebody123
        Password
                INPUT...abc123
        SUBMIT BUTTON
        ...or register_link

## device dashboard-webapp-vert-x UI

        http://localhost:8081/

        30 device updates per second
        
        Trends  / Aggregate values
        City	Volume                  
        aaaa    bbbbbb

        Public ranking (last 24 hours):
        Name	From	Volume
        xxxx     yyyyy   zzzz
        aaaa     bbbbb   cccc

# ELASTIC, DISTRIBUTED INFRASTRUCTURE

        Secure Vert.x RxJava HTTP Web Server 

        SCALABLE Vertx HTTP Server, java router class code:
        Router router = Router.router(vertx);
        // (...)                              
        vertx.createHttpServer()
                .requestHandler(router)             
                .listen(8080);
        ...
        BodyHandler bodyHandler = BodyHandler.create();             
        router.post().handler(bodyHandler);                          
        router.put().handler(bodyHandler);

        String prefix = "/api/v1";

        router.post(prefix + "/register").handler(this::register);   
        router.post(prefix + "/token").handler(this::token);
        // (...) defines jwtHandler, more later

        router.get(prefix + "/:username/:year/:month")               
                .handler(jwtHandler)                                       
                .handler(this::checkUser)
                .handler(this::monthlySteps);
        // (...)

# generating RSA 256 keys

        #!/bin/bash
        openssl genrsa -out private.pem 2048
        openssl pkcs8 -topk8 -inform PEM -in private.pem -out private_key.pem -nocrypt
        openssl rsa -in private.pem -outform PEM -pubout -out public_key.pem

        class CryptoHelper:
        static String publicKey() throws IOException {} read public key
        static String privateKey() throws IOException {...} read private key
        private static String read(String file) throws IOException {...} read file content

# create JWT handler:
        String publicKey = CryptoHelper.publicKey();
        String privateKey = CryptoHelper.privateKey();

        jwtAuth = JWTAuth.create(vertx, new JWTAuthOptions()          ❶
        .addPubSecKey(new PubSecKeyOptions()
        .setAlgorithm("RS256")
        .setBuffer(publicKey))
        .addPubSecKey(new PubSecKeyOptions()
        .setAlgorithm("RS256")
        .setBuffer(privateKey)));

        JWTAuthHandler jwtHandler = JWTAuthHandler.create(jwtAuth);
# install JWT handler in Router; CORS support can be installed in the Vert.x Router
        router.get(prefix + "/:username/:year/:month")
                .handler(jwtHandler)                           
                .handler(this::checkUser)
                .handler(this::getDevData);
# Issue JWT token with Vert.x: webclient.sendToken(context,...)

        private String makeJwtToken(String username, String deviceId) {
        JsonObject claims = new JsonObject()                         
                .put("deviceId", deviceId);
        JWTOptions jwtOptions = new JWTOptions()
                .setAlgorithm("RS256")
                .setExpiresInMinutes(10_080) // 7 days
                .setIssuer("dev-api")                                
                .setSubject(username);
        return jwtAuth.generateToken(claims, jwtOptions);
}