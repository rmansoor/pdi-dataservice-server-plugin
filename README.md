PDI DATA SERVICE PLUGIN
========================================

Objective
---------

The goal of this project is to expose transformations as a data sources and allow querying rows from a JDBC client


How to build
--------------

PDI DATA SERVICE PLUGIN uses the maven framework. 


#### Pre-requisites for building the project:
* Maven, version 3+
* Java JDK 11
* This [settings.xml](https://raw.githubusercontent.com/pentaho/maven-parent-poms/master/maven-support-files/settings.xml) in your <user-home>/.m2 directory

#### Building it

This is a maven project, and to build it use the following command

```
$ mvn clean install
```
Optionally you can specify -Drelease to trigger obfuscation and/or uglification (as needed)

Optionally you can specify -Dmaven.test.skip=true to skip the tests (even though
you shouldn't as you know)

The build result will be a Pentaho package located in ```target```.

#### Running the tests

__Unit tests__

This will run all unit tests in the project (and sub-modules). To run integration tests as well, see Integration Tests below.

```
$ mvn test
```

If you want to remote debug a single java unit test (default port is 5005):

```
$ cd core
$ mvn test -Dtest=<<YourTest>> -Dmaven.surefire.debug
```

__Integration tests__

In addition to the unit tests, there are integration tests that test cross-module operation. This will run the integration tests.

```
$ mvn verify -DrunITs
```

To run a single integration test:

```
$ mvn verify -DrunITs -Dit.test=<<YourIT>>
```

To run a single integration test in debug mode (for remote debugging in an IDE) on the default port of 5005:

```
$ mvn verify -DrunITs -Dit.test=<<YourIT>> -Dmaven.failsafe.debug
```

To skip test

```
$ mvn clean install -DskipTests
```

To get log as text file

```
$ mvn clean install test >log.txt
```


__IntelliJ__

* Don't use IntelliJ's built-in maven. Make it use the same one you use from the commandline.
  * Project Preferences -> Build, Execution, Deployment -> Build Tools -> Maven ==> Maven home directory


Sequence Diagram

graph TB
    %% ETL Design Phase
    subgraph "ETL Design Phase"
        A[ETL Designer in Spoon] --> B[Create Transformation]
        B --> C[Mark Step as Data Service]
        C --> D[Define Virtual Table Schema]
        D --> E[Save to Repository]
        E --> F[MetaStore Mapping]
    end

    %% Server Setup
    subgraph "Server Environment"
        G[DI Server] --> H[Repository Connection]
        H --> I[Data Service Registry]
        F --> I
    end

    %% Client Request Phase
    subgraph "Client Request Phase"
        J[JDBC Client] --> K[SQL Query]
        K --> L[Data Service Connection]
        L --> M[Query Parser]
    end

    %% Query Processing Engine
    subgraph "Query Processing Engine"
        M --> N[Table Name Resolution]
        N --> O[Load Service Transformation]
        O --> P[SQL Analysis]
        P --> Q[Generate Query Transformation]
        Q --> R[Apply Optimizations]
    end

    %% Optimization Layer
    subgraph "Optimization Layer"
        R --> S[Parameter Pushdown]
        R --> T[Filter Pushdown]
        R --> U[Aggregation Pushdown]
        R --> V[Sort Pushdown]
    end

    %% Execution Phase
    subgraph "Execution Phase"
        S --> W[Service Transformation Execution]
        T --> W
        U --> W
        V --> W
        W --> X[Data Processing Steps]
        X --> Y[Generate Rows]
        Y --> Z[Inject into Query Transformation]
        Z --> AA[Apply SQL Operations]
        AA --> BB[Filter/Group/Sort/Aggregate]
        BB --> CC[Result Set Generation]
    end

    %% Response Phase
    subgraph "Response Phase"
        CC --> DD[JDBC Result Set]
        DD --> EE[Return to Client]
        EE --> FF[Client Application Display]
    end

    %% Data Sources
    subgraph "Data Sources"
        GG[Database] --> X
        HH[Files] --> X
        II[Web Services] --> X
        JJ[Other Sources] --> X
    end

    %% Supporting Components
    subgraph "Supporting Components"
        KK[Security Manager]
        LL[Cache Manager]
        MM[Monitoring/Logging]
        NN[Configuration]
    end

    %% Connect supporting components
    I --> KK
    W --> LL
    W --> MM
    G --> NN

    %% Styling
    classDef designPhase fill:#e1f5fe
    classDef serverPhase fill:#f3e5f5
    classDef clientPhase fill:#e8f5e8
    classDef processingPhase fill:#fff3e0
    classDef optimizationPhase fill:#fce4ec
    classDef executionPhase fill:#e0f2f1
    classDef responsePhase fill:#f1f8e9
    classDef dataSource fill:#fff8e1
    classDef support fill:#fafafa

    class A,B,C,D,E,F designPhase
    class G,H,I serverPhase
    class J,K,L,M clientPhase
    class N,O,P,Q,R processingPhase
    class S,T,U,V optimizationPhase
    class W,X,Y,Z,AA,BB,CC executionPhase
    class DD,EE,FF responsePhase
    class GG,HH,II,JJ dataSource
    class KK,LL,MM,NN support

