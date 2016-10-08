# Your Code As a Crime Scene

Mining/Analytics data repo template for any git repo using [Code Maat Tool](https://github.com/adamtornhill/code-maat) from the amazing book [Your Code as a Crime Scene](https://www.amazon.com/Your-Code-Crime-Scene-Bottlenecks/dp/1680500384).

# HOTSPOTS ANALYSES:

## What does Hotspots mean here?

	Hotspots here are basically a result of a combination of:

		- Change Frequencies/Effort (Number of Revisions) with
		- Code Complexity/Size (Number of Lines)

		** Both metrics are collected per file based.


"The resulting output ('*-hotspots.csv') is sorted on change frequencies first - the best predictor of problems - and size (number of lines) second."


## Step by Step to generate the data:

1. Git clone code-maat
 ```
 $ cd ~/workspace
 $ git clone git@github.com:adamtornhill/code-maat.git
 ```

2. Generate the Standalone JAR file for code-maat using leiningen
 ```
 $ brew install leiningen
 $ lein help
 $ cd ~/workspace/code-maat
 $ lein uberjar
 $ cp target/code-maat-1.0-SNAPSHOT-standalone.jar ~/workspace
 ```

3. Add alias for maat
 ```
 $ java -jar ~/workspace/code-maat-1.0-SNAPSHOT-standalone.jar
 $ alias maat = 'java -jar ~/workspace/code-maat-1.0-SNAPSHOT-standalone.jar'
 ```

4. Install cloc to count number of lines per file
 ```
 $ brew install cloc
 ```

5. Choose a data interval for the analyses and generate the evo file for the period:
 ```
 $ cd ~/workspace/your-repo
 $ git log --pretty=format:'[%h] %aN %ad %s' --date=short --numstat --before=2015-09-01 --after=2014-10-01  > ~/src/test-data/your-repo-2015-09-01-evo.log
 ```

6. Checkout the repo to the most recent day of analyses and collect the number of lines data using cloc:
 ```
 $ cd ~/workspace/your-repo
 $ git checkout `git rev-list -n 1 --before="2015-09-01" master`
 $ cloc ./ --by-file --csv --quiet --report-file=~/src/test-data/your-repo-2015-09-01-lines.csv
 ```

7. Generate the summary and revisions CSV files using maat:
 ```
 $ maat -l your-repo-2015-09-01-evo.log -c git -a summary > ~/src/test-data/your-repo-2015-09-01-summary.csv
 $ maat -l your-repo-2015-09-01-evo.log -c git -a revisions > ~/src/test-data/your-repo-2015-09-01-revisions.csv
 ```

8. Merge the change frequencies and the number of lines data file to create the hotspots file:
 ```
 $ python scripts/merge_comp_freqs.py your-repo-2015-09-01-revisions.csv your-repo-2015-09-01-lines.csv > your-repo-2015-09-01-hotspots.csv
 ```

9. Generate the JSON file for D3.js visualization:
 ```
 $ python scripts/csv_as_enclosure_json.py --structure=your-repo-2013-12-31-lines.csv --weights=your-repo-2013-12-31-hotspots.csv > your-repo-2013-12-31-d3js-data.json
 ```

10. Copy the D3.js folder and change the HTML template file for D3.js to load our local .json file:
 ```
 $ cp -r ~/workspace/code-maat-mining-data/samples/hibernate/d3 .
 $ cp  ~/workspace/code-maat-mining-data/samples/hibernate/hibzoomable.html .
 $ mv hibzoomable.html webzoomable.html
 $ vim webzoomable.html
 ```

11. Start a local HTTP server and visualize your hotspots data:
 ```
 # in ~/workspace/code-maat-mining-data/your-repo-hotspots/2015-09-01
 $ python -m SimpleHTTPServer 8888
	Serving HTTP on 0.0.0.0 port 8888 ...

 $ open http://localhost:8888/webzoomable.html
 ```

## How can you use the Hotspots analyses?

	Using hotspots as a Guide:

	"Similar to forensic psychology methods, code offender profiling techniques help us narrow down the search area within a codebase."

	That means we can focus our human expertise on smaller, more focused parts of the system. This opens up a number of possibilities:

	1. *Prioritize design issues*: I started this investigation because I needed a way to plan and prioritize improvements to a legacy system. Some of the suggested improvements were redesigns that cost several weeks of intense work. I had to ensure we spent time on improvements that actually helped future development efforts. Hotspots, indicating complex code that changes frequently, proved to be great candidates. With the material in this chapter, you have the basis to do the same analysis yourself.

	2. *Perform code reviews*: Code reviews have high defect-removal rates. A code review done right is also time-consuming. Because hotspots make good predictors of buggy code (see ​The Relationship Between Hotspots and Defects​), they identify the parts where reviews would be a good investment of time.

	3. *Improve communication*: On a recent project I used the results of a hotspot analysis to communicate with the test leader. That project had several skilled testers. They often spent the end of each iteration on exploratory testing. The test leader wanted a simple way to identify feature areas that could benefit from such additional testing. Hotspots make a perfect starting point for these kinds of tests.


#TODO:

	- Code City to Explory the geography of our code

