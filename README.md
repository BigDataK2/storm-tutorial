# storm-tutorial - distributed wc

# ABOUT

Here's a funny sort of Hello World for distributed Java programming with Storm. This Storm topology performs a word count over the complete Sherlock Holmes, and displays the number of occurrences of the word `Watson`.

# EXAMPLE

Starting with a lot of text in an input directory:

```
$ tree resources/sherlock-holmes/
resources/sherlock-holmes/
├── agrange.txt
├── b-p_plan.txt
├── bascombe.txt
├── beryl.txt
├── blanced.txt
├── blkpeter.txt
├── bluecar.txt
├── cardbox.txt
├── caseide.txt
├── charles.txt
├── copper.txt
├── creeping.txt
├── crookman.txt
├── danceman.txt
├── devilsf.txt
├── doyle-adventures-380.txt
...
```

We write a Storm topology to divide up the work for counting words.

```
$ tail -n 14 src/main/java/us/yellosoft/storm/tutorial/WordCountTopology.java
  public static void main(String[] args) throws Exception {
    TopologyBuilder builder = new TopologyBuilder();
    builder.setSpout("text-spout", new TextSpout(), 1);
    builder.setBolt("word-splitter", new WordSplitter(), 4).shuffleGrouping("text-spout");
    builder.setBolt("word-counter", new WordCount(), 8).fieldsGrouping("word-splitter", new Fields("word"));

    Config config = new Config();
    config.setDebug(true);
    config.setMaxTaskParallelism(3);

    LocalCluster cluster = new LocalCluster();
    cluster.submitTopology("word-count", config, builder.createTopology());
  }
}
```

We compile and run the job,

```
$ gradle build
$ storm jar build/libs/storm-tutorial.jar us.yellosoft.storm.tutorial.WordCountTopology
...
69856 [Thread-10-word-counter] INFO  backtype.storm.daemon.task - Emitting: word-counter default [pointed, 22]
...
Control+C
```

We use the data generated by the Storm topology and notice a funny pattern:

```
$ storm \
  jar \
  build/libs/storm-tutorial.jar \
  us.yellosoft.storm.tutorial.WordCountTopology \
  | grep 'Emitting: word-counter default \[Watson.*, ' \
  | sed 's/^.*default\s*//'
...
 [Watson!", 9]
 [Watson,, 76]
 [Watson!", 10]
 [Watson,", 49]
 [Watson,, 77]
 [Watson, 38]
 [Watson,, 78]
 [Watson,", 50]
 [Watson., 52]
 [Watson,", 51]
 [Watson,", 52]
 [Watson,, 79]
 [Watson,, 80]
 [Watson,", 53]
 [Watson,", 54]
 [Watson,, 81]
 [Watson,, 82]
 [Watson,", 55]
 [Watson?", 20]
 [Watson,, 83]
 [Watson, 39]
 [Watson,, 84]
 [Watson,, 85]
 [Watson,, 86]
 [Watson,, 87]
 [Watson,, 88]
 [Watson,", 56]
 [Watson,", 57]
 [Watson,", 58]
 [Watson,", 59]
 [Watson,", 60]
 [Watson,", 61]
 [Watson!, 36]
 [Watson., 53]
 [Watson,, 89]
 [Watson,, 90]
 [Watson,, 91]
 [Watson,, 92]
 [Watson,, 93]
 [Watson,, 94]
 ...
 Control+C
```

This example is tested to successfully compile and run in Mac OS X, currently the only OS for which installing Storm isn't a painful exercise in *yak shaving*.

# REQUIREMENTS

* [JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 1.7+
* [Gradle](http://gradle.org/) 2.1+
* [Storm](https://storm.apache.org/) 0.9.3+
* `make`
* `sh`
* Ensure the `storm` script is in `$PATH` (test with `which storm`).

## Optional

* `tree`
* `grep`
* `sed`
* `tail`
* [Sonar](http://www.sonarqube.org/)
* [Ruby](https://www.ruby-lang.org/) 1.9+
* [Guard](http://guardgem.org/) 1.8.2+

Use `bundle` to install Guard.

## Mac OS X

Mac comes with JDK, `sh`, `grep`, `sed`, and `tail` by default. The remainder can be obtained thusly:

1. Install [Xcode](https://developer.apple.com/xcode/) via `App Store.app`.
2. Launch `Xcode.app`, install the Xcode command line tools.
3. Install [Homebrew](http://brew.sh/).
4. Launch `Terminal.app`, run `brew install maven storm`.

Optionally, run `brew install tree sonar sonar-runner`.

## Huh?

Either `git clone https://github.com/mcandre/storm-tutorial.git`, or manually download and extract the [ZIP](https://github.com/mcandre/storm-tutorial/archive/master.zip)ball.

Consider forking [storm-tutorial](https://github.com/mcandre/storm-tutorial) and amending the README to help other Stormers along the way.

# CREDITS

* [Apache Storm Starter](https://github.com/apache/storm/tree/master/examples/storm-starter) - Source code for `WordCountTopology.java`
