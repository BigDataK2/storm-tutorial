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
$ mvn package
$ storm jar target/storm-tutorial-0.0.0.jar us.yellosoft.storm.tutorial.WordCountTopology
...
69856 [Thread-10-word-counter] INFO  backtype.storm.daemon.task - Emitting: word-counter default [pointed, 22]
...
```

We use the data generated by the Storm topology and notice a funny pattern:

```
$ storm jar target/storm-tutorial-0.0.0.jar us.yellosoft.storm.tutorial.WordCountTopology | grep 'Emitting: word-counter default \[Watson.*, '
...
14772 [Thread-14-word-counter] INFO  backtype.storm.daemon.task - Emitting: word-counter default [Watson,", 67]
14783 [Thread-10-word-counter] INFO  backtype.storm.daemon.task - Emitting: word-counter default [Watson's, 2]
14786 [Thread-14-word-counter] INFO  backtype.storm.daemon.task - Emitting: word-counter default [Watson,, 96]
14807 [Thread-12-word-counter] INFO  backtype.storm.daemon.task - Emitting: word-counter default [Watson., 55]
14810 [Thread-12-word-counter] INFO  backtype.storm.daemon.task - Emitting: word-counter default [Watson., 56]
14811 [Thread-12-word-counter] INFO  backtype.storm.daemon.task - Emitting: word-counter default [Watson., 57]
14820 [Thread-14-word-counter] INFO  backtype.storm.daemon.task - Emitting: word-counter default [Watson,, 97]
14874 [Thread-12-word-counter] INFO  backtype.storm.daemon.task - Emitting: word-counter default [Watson., 58]
14886 [Thread-10-word-counter] INFO  backtype.storm.daemon.task - Emitting: word-counter default [Watson?", 21]
14893 [Thread-12-word-counter] INFO  backtype.storm.daemon.task - Emitting: word-counter default [Watson., 59]
14900 [Thread-10-word-counter] INFO  backtype.storm.daemon.task - Emitting: word-counter default [Watson?", 22]
14905 [Thread-14-word-counter] INFO  backtype.storm.daemon.task - Emitting: word-counter default [Watson,, 98]
14905 [Thread-14-word-counter] INFO  backtype.storm.daemon.task - Emitting: word-counter default [Watson,, 99]
14906 [Thread-14-word-counter] INFO  backtype.storm.daemon.task - Emitting: word-counter default [Watson,, 100]
14906 [Thread-14-word-counter] INFO  backtype.storm.daemon.task - Emitting: word-counter default [Watson,, 101]
14906 [Thread-14-word-counter] INFO  backtype.storm.daemon.task - Emitting: word-counter default [Watson,", 68]
14906 [Thread-14-word-counter] INFO  backtype.storm.daemon.task - Emitting: word-counter default [Watson,", 69]
14912 [Thread-14-word-counter] INFO  backtype.storm.daemon.task - Emitting: word-counter default [Watson,, 102]
...
```

This example is tested to successfully compile and run in Mac OS X, currently the only OS for which installing Storm isn't a painful exercise in *yak shaving*.

# REQUIREMENTS

* [JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 6+
* [Maven](http://maven.apache.org/) 3+
* [Storm](https://storm.apache.org/) 0.9.3+
* `make`
* `sh`
* Ensure the `storm` script is in `$PATH` (test with `which storm`).

## Optional

* `tree`
* `grep`
* `tail`
* [Ruby](https://www.ruby-lang.org/) 1.9+
* [Guard](http://guardgem.org/) 1.8.2+

Use `bundle` to install Guard.

## Mac OS X

Mac comes with JDK, `sh`, `grep`, and `tail` by default. The remainder can be obtained thusly:

1. Install [Xcode](https://developer.apple.com/xcode/) via `App Store.app`.
2. Launch `Xcode.app`, install the Xcode command line tools.
3. Install [Homebrew](http://brew.sh/).
4. Launch `Terminal.app`, run `brew install maven storm`.

Optionally, run `brew install tree`.

## Huh?

Either `git clone https://github.com/mcandre/storm-tutorial.git`, or manually download and extract the [ZIP](https://github.com/mcandre/storm-tutorial/archive/master.zip)ball.

Consider forking [storm-tutorial](https://github.com/mcandre/storm-tutorial) and amending the README to help other Stormers along the way.

# CREDITS

* [Apache Storm Starter](https://github.com/apache/storm/tree/master/examples/storm-starter) - Source code for `WordCountTopology.java`
