

```shell
2018-12-24 22:45:30,418 WARN  client.QuorumJournalManager (QuorumCall.java:waitFor(134)) - Waited 19032 ms (timeout=20000 ms) for a response for sendE
dits. Succeeded so far: [10.10.22.3:8485]. Exceptions so far: [10.10.22.2:8485: Journal disabled until next roll]
2018-12-24 22:45:31,688 FATAL namenode.FSEditLog (JournalSet.java:mapJournalsAndReportErrors(398)) - Error: flush failed for required journal (Journal
AndStream(mgr=QJM to [10.10.22.3:8485, 10.10.22.2:8485, 10.10.22.4:8485], stream=QuorumOutputStream starting at txid 35387466))
java.io.IOException: Timed out waiting 20000ms for a quorum of nodes to respond.
```



改这个

```xml


<property>
        <name>dfs.qjournal.write-txns.timeout.ms</name>
        <value>60000</value>
</property>
```

