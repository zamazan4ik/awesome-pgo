# PostgreSQL: LTO with PGO results

Here I want to share my results with LTO + PGO PostgreSQL optimization.

## Test environment

* Fedora 38
* Linux kernel 6.2.15
* AMD Ryzen 9 5800x
* 48 Gib RAM
* SSD Samsung 980 Pro 2 Tib
* Clang 16 (from the Fedora repositories). I use Clang just because I prefer LLVM-based tooling
* PostgreSQL version: latest commit (`51b2c08798867cb9788090704b37c4698b456240` commit)

## Tested configurations

I have tested the following PostgreSQL configurations (with corresponding `CFLAGS` and `CXXFLAGS`):

* Release: `-O3`
* Release with LTO: `-O3 -flto`
* Release with LTO and machine-specific optimization: `-O3 -flto -march=native`
* Release with LTO and AutoFDO: `-O3 -flto -fprofile-sample-use` (+ `perf record ` report)
* Release with LTO and PGO: `-O3 -flto -fprofile-instr-use`

## Benchmark

I use [ClickBench](https://github.com/ClickHouse/ClickBench/tree/main/postgresql) as a PostgreSQL benchmark. I am quite familiar with this bench quite already and it's proven by multiple people. The same tests could be repeated with `pgbench` as well somewhere later.

I understand ClickBench does not represent a "fully tuned" PostgreSQL instance but I have no aim to do this here. I just want to show you applicability of LTO and PGO optimization even for suboptimal PostgreSQL configurations.

So a default PostgreSQL configuration I use a provided by `initdb -D data` except `shared_buffers` - I increased them to 2048 Mib. That's not an important detail - I am sure the same results could be achieved without this tweak - I just did it because... why not?

## Results

Here are results of running ClickBench test suite on different configurations. All configurations are benchmarked on the same machine, with the same PostgreSQL configuration, multiple times, etc.

Also, I shared results for Instrumentation mode so you would be able to estimate how slow PostgreSQL is in Instrumentation mode for PGO.

Another additional results are for AutoFDO. For some unknown yet reasons AutoFDO does not bring performance improvements. I have at least two guesses for this:

* I do something wrong :) (but I follow instructions from Clang manual and AutoFDO repo)
* Since I have a setup without LBR/BRS support (`perf` says me so on my machine) - I got worse results since according to the AutoFDO papers it likes LBR/BRS setups.

So maybe in your case you could try to use AutoFDO as well if you have a correspoding hardware.

<details>
  <summary>Release</summary>

  ```
  3
SELECT COUNT(*) FROM hits;
Time: 13092,144 ms (00:13,092)
Time: 13171,905 ms (00:13,172)
Time: 13070,657 ms (00:13,071)
3
SELECT COUNT(*) FROM hits WHERE AdvEngineID <> 0;
Time: 14713,633 ms (00:14,714)
Time: 15082,173 ms (00:15,082)
Time: 15173,257 ms (00:15,173)
3
SELECT SUM(AdvEngineID), COUNT(*), AVG(ResolutionWidth) FROM hits;
Time: 15149,016 ms (00:15,149)
Time: 15669,502 ms (00:15,670)
Time: 15892,338 ms (00:15,892)
3
SELECT AVG(UserID) FROM hits;
Time: 13326,762 ms (00:13,327)
Time: 13619,480 ms (00:13,619)
Time: 13564,382 ms (00:13,564)
3
SELECT COUNT(DISTINCT UserID) FROM hits;
Time: 43551,056 ms (00:43,551)
Time: 43981,082 ms (00:43,981)
Time: 44385,471 ms (00:44,385)
3
SELECT COUNT(DISTINCT SearchPhrase) FROM hits;
Time: 76007,950 ms (01:16,008)
Time: 76441,718 ms (01:16,442)
Time: 76536,060 ms (01:16,536)
3
SELECT MIN(EventDate), MAX(EventDate) FROM hits;
Time: 13158,355 ms (00:13,158)
Time: 13482,569 ms (00:13,483)
Time: 13439,202 ms (00:13,439)
3
SELECT AdvEngineID, COUNT(*) FROM hits WHERE AdvEngineID <> 0 GROUP BY AdvEngineID ORDER BY COUNT(*) DESC;
Time: 14730,010 ms (00:14,730)
Time: 15337,586 ms (00:15,338)
Time: 15205,456 ms (00:15,205)
3
SELECT RegionID, COUNT(DISTINCT UserID) AS u FROM hits GROUP BY RegionID ORDER BY u DESC LIMIT 10;
Time: 32643,024 ms (00:32,643)
Time: 32979,626 ms (00:32,980)
Time: 32947,257 ms (00:32,947)
3
SELECT RegionID, SUM(AdvEngineID), COUNT(*) AS c, AVG(ResolutionWidth), COUNT(DISTINCT UserID) FROM hits GROUP BY RegionID ORDER BY c DESC LIMIT 10;
Time: 38208,999 ms (00:38,209)
Time: 38246,867 ms (00:38,247)
Time: 38357,603 ms (00:38,358)
3
SELECT MobilePhoneModel, COUNT(DISTINCT UserID) AS u FROM hits WHERE MobilePhoneModel <> '' GROUP BY MobilePhoneModel ORDER BY u DESC LIMIT 10;
Time: 16463,870 ms (00:16,464)
Time: 16945,578 ms (00:16,946)
Time: 16864,566 ms (00:16,865)
3
SELECT MobilePhone, MobilePhoneModel, COUNT(DISTINCT UserID) AS u FROM hits WHERE MobilePhoneModel <> '' GROUP BY MobilePhone, MobilePhoneModel ORDER BY u DESC LIMIT 10;
Time: 17248,661 ms (00:17,249)
Time: 17616,117 ms (00:17,616)
Time: 17661,647 ms (00:17,662)
3
SELECT SearchPhrase, COUNT(*) AS c FROM hits WHERE SearchPhrase <> '' GROUP BY SearchPhrase ORDER BY c DESC LIMIT 10;
Time: 27742,611 ms (00:27,743)
Time: 27897,742 ms (00:27,898)
Time: 28063,128 ms (00:28,063)
3
SELECT SearchPhrase, COUNT(DISTINCT UserID) AS u FROM hits WHERE SearchPhrase <> '' GROUP BY SearchPhrase ORDER BY u DESC LIMIT 10;
Time: 26346,740 ms (00:26,347)
Time: 26648,066 ms (00:26,648)
Time: 27016,050 ms (00:27,016)
3
SELECT SearchEngineID, SearchPhrase, COUNT(*) AS c FROM hits WHERE SearchPhrase <> '' GROUP BY SearchEngineID, SearchPhrase ORDER BY c DESC LIMIT 10;
Time: 31352,938 ms (00:31,353)
Time: 31729,371 ms (00:31,729)
Time: 31685,647 ms (00:31,686)
3
SELECT UserID, COUNT(*) FROM hits GROUP BY UserID ORDER BY COUNT(*) DESC LIMIT 10;
Time: 54735,080 ms (00:54,735)
Time: 55608,377 ms (00:55,608)
Time: 56038,906 ms (00:56,039)
3
SELECT UserID, SearchPhrase, COUNT(*) FROM hits GROUP BY UserID, SearchPhrase ORDER BY COUNT(*) DESC LIMIT 10;
Time: 52340,633 ms (00:52,341)
Time: 53301,715 ms (00:53,302)
Time: 52845,217 ms (00:52,845)
3
SELECT UserID, SearchPhrase, COUNT(*) FROM hits GROUP BY UserID, SearchPhrase LIMIT 10;
Time: 41311,127 ms (00:41,311)
Time: 41874,873 ms (00:41,875)
Time: 42109,233 ms (00:42,109)
3
SELECT UserID, extract(minute FROM EventTime) AS m, SearchPhrase, COUNT(*) FROM hits GROUP BY UserID, m, SearchPhrase ORDER BY COUNT(*) DESC LIMIT 10;
Time: 58782,227 ms (00:58,782)
Time: 59007,424 ms (00:59,007)
Time: 58909,482 ms (00:58,909)
3
SELECT UserID FROM hits WHERE UserID = 435090932899640449;
Time: 13447,487 ms (00:13,447)
Time: 13480,931 ms (00:13,481)
Time: 13405,165 ms (00:13,405)
3
SELECT COUNT(*) FROM hits WHERE URL LIKE '%google%';
Time: 15449,790 ms (00:15,450)
Time: 16271,898 ms (00:16,272)
Time: 15931,388 ms (00:15,931)
3
SELECT SearchPhrase, MIN(URL), COUNT(*) AS c FROM hits WHERE URL LIKE '%google%' AND SearchPhrase <> '' GROUP BY SearchPhrase ORDER BY c DESC LIMIT 10;
Time: 17440,006 ms (00:17,440)
Time: 17589,659 ms (00:17,590)
Time: 17692,300 ms (00:17,692)
3
SELECT SearchPhrase, MIN(URL), MIN(Title), COUNT(*) AS c, COUNT(DISTINCT UserID) FROM hits WHERE Title LIKE '%Google%' AND URL NOT LIKE '%.google.%' AND SearchPhrase <> '' GROUP BY SearchPhrase ORDER BY c DESC LIMIT 10;
Time: 17726,333 ms (00:17,726)
Time: 18212,388 ms (00:18,212)
Time: 18158,746 ms (00:18,159)
3
SELECT * FROM hits WHERE URL LIKE '%google%' ORDER BY EventTime LIMIT 10;
Time: 15511,494 ms (00:15,511)
Time: 16029,887 ms (00:16,030)
Time: 16052,541 ms (00:16,053)
3
SELECT SearchPhrase FROM hits WHERE SearchPhrase <> '' ORDER BY EventTime LIMIT 10;
Time: 15044,900 ms (00:15,045)
Time: 15766,035 ms (00:15,766)
Time: 15517,740 ms (00:15,518)
3
SELECT SearchPhrase FROM hits WHERE SearchPhrase <> '' ORDER BY SearchPhrase LIMIT 10;
Time: 15228,293 ms (00:15,228)
Time: 15725,522 ms (00:15,726)
Time: 15719,025 ms (00:15,719)
3
SELECT SearchPhrase FROM hits WHERE SearchPhrase <> '' ORDER BY EventTime, SearchPhrase LIMIT 10;
Time: 15014,271 ms (00:15,014)
Time: 15679,945 ms (00:15,680)
Time: 15559,037 ms (00:15,559)
3
SELECT CounterID, AVG(length(URL)) AS l, COUNT(*) AS c FROM hits WHERE URL <> '' GROUP BY CounterID HAVING COUNT(*) > 100000 ORDER BY l DESC LIMIT 25;
Time: 18541,336 ms (00:18,541)
Time: 19268,487 ms (00:19,268)
Time: 18977,056 ms (00:18,977)
3
SELECT REGEXP_REPLACE(Referer, '^https?://(?:www.)?([^/]+)/.*$', '1') AS k, AVG(length(Referer)) AS l, COUNT(*) AS c, MIN(Referer) FROM hits WHERE Referer <> '' GROUP BY k HAVING COUNT(*) > 100000 ORDER BY l DESC LIMIT 25;
Time: 70922,401 ms (01:10,922)
Time: 70946,553 ms (01:10,947)
Time: 70391,945 ms (01:10,392)
3
SELECT SUM(ResolutionWidth), SUM(ResolutionWidth + 1), SUM(ResolutionWidth + 2), SUM(ResolutionWidth + 3), SUM(ResolutionWidth + 4), SUM(ResolutionWidth + 5), SUM(ResolutionWidth + 6), SUM(ResolutionWidth + 7), SUM(ResolutionWidth + 8), SUM(ResolutionWidth + 9), SUM(ResolutionWidth + 10), SUM(ResolutionWidth + 11), SUM(ResolutionWidth + 12), SUM(ResolutionWidth + 13), SUM(ResolutionWidth + 14), SUM(ResolutionWidth + 15), SUM(ResolutionWidth + 16), SUM(ResolutionWidth + 17), SUM(ResolutionWidth + 18), SUM(ResolutionWidth + 19), SUM(ResolutionWidth + 20), SUM(ResolutionWidth + 21), SUM(ResolutionWidth + 22), SUM(ResolutionWidth + 23), SUM(ResolutionWidth + 24), SUM(ResolutionWidth + 25), SUM(ResolutionWidth + 26), SUM(ResolutionWidth + 27), SUM(ResolutionWidth + 28), SUM(ResolutionWidth + 29), SUM(ResolutionWidth + 30), SUM(ResolutionWidth + 31), SUM(ResolutionWidth + 32), SUM(ResolutionWidth + 33), SUM(ResolutionWidth + 34), SUM(ResolutionWidth + 35), SUM(ResolutionWidth + 36), SUM(ResolutionWidth + 37), SUM(ResolutionWidth + 38), SUM(ResolutionWidth + 39), SUM(ResolutionWidth + 40), SUM(ResolutionWidth + 41), SUM(ResolutionWidth + 42), SUM(ResolutionWidth + 43), SUM(ResolutionWidth + 44), SUM(ResolutionWidth + 45), SUM(ResolutionWidth + 46), SUM(ResolutionWidth + 47), SUM(ResolutionWidth + 48), SUM(ResolutionWidth + 49), SUM(ResolutionWidth + 50), SUM(ResolutionWidth + 51), SUM(ResolutionWidth + 52), SUM(ResolutionWidth + 53), SUM(ResolutionWidth + 54), SUM(ResolutionWidth + 55), SUM(ResolutionWidth + 56), SUM(ResolutionWidth + 57), SUM(ResolutionWidth + 58), SUM(ResolutionWidth + 59), SUM(ResolutionWidth + 60), SUM(ResolutionWidth + 61), SUM(ResolutionWidth + 62), SUM(ResolutionWidth + 63), SUM(ResolutionWidth + 64), SUM(ResolutionWidth + 65), SUM(ResolutionWidth + 66), SUM(ResolutionWidth + 67), SUM(ResolutionWidth + 68), SUM(ResolutionWidth + 69), SUM(ResolutionWidth + 70), SUM(ResolutionWidth + 71), SUM(ResolutionWidth + 72), SUM(ResolutionWidth + 73), SUM(ResolutionWidth + 74), SUM(ResolutionWidth + 75), SUM(ResolutionWidth + 76), SUM(ResolutionWidth + 77), SUM(ResolutionWidth + 78), SUM(ResolutionWidth + 79), SUM(ResolutionWidth + 80), SUM(ResolutionWidth + 81), SUM(ResolutionWidth + 82), SUM(ResolutionWidth + 83), SUM(ResolutionWidth + 84), SUM(ResolutionWidth + 85), SUM(ResolutionWidth + 86), SUM(ResolutionWidth + 87), SUM(ResolutionWidth + 88), SUM(ResolutionWidth + 89) FROM hits;
Time: 36112,948 ms (00:36,113)
Time: 36164,319 ms (00:36,164)
Time: 36412,808 ms (00:36,413)
3
SELECT SearchEngineID, ClientIP, COUNT(*) AS c, SUM(IsRefresh), AVG(ResolutionWidth) FROM hits WHERE SearchPhrase <> '' GROUP BY SearchEngineID, ClientIP ORDER BY c DESC LIMIT 10;
Time: 22416,068 ms (00:22,416)
Time: 23233,670 ms (00:23,234)
Time: 23284,737 ms (00:23,285)
3
SELECT WatchID, ClientIP, COUNT(*) AS c, SUM(IsRefresh), AVG(ResolutionWidth) FROM hits WHERE SearchPhrase <> '' GROUP BY WatchID, ClientIP ORDER BY c DESC LIMIT 10;
Time: 22448,181 ms (00:22,448)
Time: 23023,206 ms (00:23,023)
Time: 22911,552 ms (00:22,912)
3
SELECT WatchID, ClientIP, COUNT(*) AS c, SUM(IsRefresh), AVG(ResolutionWidth) FROM hits GROUP BY WatchID, ClientIP ORDER BY c DESC LIMIT 10;
Time: 195240,946 ms (03:15,241)
Time: 192673,286 ms (03:12,673)
Time: 192911,723 ms (03:12,912)
3
SELECT URL, COUNT(*) AS c FROM hits GROUP BY URL ORDER BY c DESC LIMIT 10;
Time: 93733,791 ms (01:33,734)
Time: 94240,237 ms (01:34,240)
Time: 94237,050 ms (01:34,237)
3
SELECT 1, URL, COUNT(*) AS c FROM hits GROUP BY 1, URL ORDER BY c DESC LIMIT 10;
Time: 93731,240 ms (01:33,731)
Time: 94597,121 ms (01:34,597)
Time: 95578,355 ms (01:35,578)
3
SELECT ClientIP, ClientIP - 1, ClientIP - 2, ClientIP - 3, COUNT(*) AS c FROM hits GROUP BY ClientIP, ClientIP - 1, ClientIP - 2, ClientIP - 3 ORDER BY c DESC LIMIT 10;
Time: 27768,100 ms (00:27,768)
Time: 28188,639 ms (00:28,189)
Time: 27692,206 ms (00:27,692)
3
SELECT URL, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND DontCountHits = 0 AND IsRefresh = 0 AND URL <> '' GROUP BY URL ORDER BY PageViews DESC LIMIT 10;
Time: 17134,389 ms (00:17,134)
Time: 17672,381 ms (00:17,672)
Time: 17481,163 ms (00:17,481)
3
SELECT Title, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND DontCountHits = 0 AND IsRefresh = 0 AND Title <> '' GROUP BY Title ORDER BY PageViews DESC LIMIT 10;
Time: 16924,369 ms (00:16,924)
Time: 17680,954 ms (00:17,681)
Time: 17455,158 ms (00:17,455)
3
SELECT URL, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND IsRefresh = 0 AND IsLink <> 0 AND IsDownload = 0 GROUP BY URL ORDER BY PageViews DESC LIMIT 10 OFFSET 1000;
Time: 15817,456 ms (00:15,817)
Time: 16441,138 ms (00:16,441)
Time: 16261,022 ms (00:16,261)
3
SELECT TraficSourceID, SearchEngineID, AdvEngineID, CASE WHEN (SearchEngineID = 0 AND AdvEngineID = 0) THEN Referer ELSE '' END AS Src, URL AS Dst, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND IsRefresh = 0 GROUP BY TraficSourceID, SearchEngineID, AdvEngineID, Src, Dst ORDER BY PageViews DESC LIMIT 10 OFFSET 1000;
Time: 14345,236 ms (00:14,345)
Time: 14657,673 ms (00:14,658)
Time: 14563,443 ms (00:14,563)
3
SELECT URLHash, EventDate, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND IsRefresh = 0 AND TraficSourceID IN (-1, 6) AND RefererHash = 3594120000172545465 GROUP BY URLHash, EventDate ORDER BY PageViews DESC LIMIT 10 OFFSET 100;
Time: 20585,834 ms (00:20,586)
Time: 20838,136 ms (00:20,838)
Time: 20780,094 ms (00:20,780)
3
SELECT WindowClientWidth, WindowClientHeight, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND IsRefresh = 0 AND DontCountHits = 0 AND URLHash = 2868770270353813622 GROUP BY WindowClientWidth, WindowClientHeight ORDER BY PageViews DESC LIMIT 10 OFFSET 10000;
Time: 20117,453 ms (00:20,117)
Time: 20464,375 ms (00:20,464)
Time: 20498,047 ms (00:20,498)
3
SELECT DATE_TRUNC('minute', EventTime) AS M, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-14' AND EventDate <= '2013-07-15' AND IsRefresh = 0 AND DontCountHits = 0 GROUP BY DATE_TRUNC('minute', EventTime) ORDER BY DATE_TRUNC('minute', EventTime) LIMIT 10 OFFSET 1000;
Time: 16289,420 ms (00:16,289)
Time: 16882,633 ms (00:16,883)
Time: 16737,045 ms (00:16,737)

  ```
</details>

<details>
  <summary>Release with LTO</summary>
  
  ```
  3
SELECT COUNT(*) FROM hits;
Time: 12957,548 ms (00:12,958)
Time: 13129,711 ms (00:13,130)
Time: 13097,653 ms (00:13,098)
3
SELECT COUNT(*) FROM hits WHERE AdvEngineID <> 0;
Time: 14784,694 ms (00:14,785)
Time: 15253,880 ms (00:15,254)
Time: 15151,476 ms (00:15,151)
3
SELECT SUM(AdvEngineID), COUNT(*), AVG(ResolutionWidth) FROM hits;
Time: 15069,227 ms (00:15,069)
Time: 15688,348 ms (00:15,688)
Time: 15560,825 ms (00:15,561)
3
SELECT AVG(UserID) FROM hits;
Time: 13335,451 ms (00:13,335)
Time: 13536,843 ms (00:13,537)
Time: 13475,600 ms (00:13,476)
3
SELECT COUNT(DISTINCT UserID) FROM hits;
Time: 43468,296 ms (00:43,468)
Time: 44101,387 ms (00:44,101)
Time: 44065,461 ms (00:44,065)
3
SELECT COUNT(DISTINCT SearchPhrase) FROM hits;
Time: 74388,772 ms (01:14,389)
Time: 75015,001 ms (01:15,015)
Time: 74454,084 ms (01:14,454)
3
SELECT MIN(EventDate), MAX(EventDate) FROM hits;
Time: 13244,773 ms (00:13,245)
Time: 13686,707 ms (00:13,687)
Time: 13385,813 ms (00:13,386)
3
SELECT AdvEngineID, COUNT(*) FROM hits WHERE AdvEngineID <> 0 GROUP BY AdvEngineID ORDER BY COUNT(*) DESC;
Time: 14710,829 ms (00:14,711)
Time: 15293,036 ms (00:15,293)
Time: 15257,434 ms (00:15,257)
3
SELECT RegionID, COUNT(DISTINCT UserID) AS u FROM hits GROUP BY RegionID ORDER BY u DESC LIMIT 10;
Time: 31505,905 ms (00:31,506)
Time: 32861,321 ms (00:32,861)
Time: 32959,106 ms (00:32,959)
3
SELECT RegionID, SUM(AdvEngineID), COUNT(*) AS c, AVG(ResolutionWidth), COUNT(DISTINCT UserID) FROM hits GROUP BY RegionID ORDER BY c DESC LIMIT 10;
Time: 38514,748 ms (00:38,515)
Time: 38569,587 ms (00:38,570)
Time: 39201,440 ms (00:39,201)
3
SELECT MobilePhoneModel, COUNT(DISTINCT UserID) AS u FROM hits WHERE MobilePhoneModel <> '' GROUP BY MobilePhoneModel ORDER BY u DESC LIMIT 10;
Time: 16482,985 ms (00:16,483)
Time: 16911,967 ms (00:16,912)
Time: 16936,766 ms (00:16,937)
3
SELECT MobilePhone, MobilePhoneModel, COUNT(DISTINCT UserID) AS u FROM hits WHERE MobilePhoneModel <> '' GROUP BY MobilePhone, MobilePhoneModel ORDER BY u DESC LIMIT 10;
Time: 16872,318 ms (00:16,872)
Time: 17534,779 ms (00:17,535)
Time: 17428,043 ms (00:17,428)
3
SELECT SearchPhrase, COUNT(*) AS c FROM hits WHERE SearchPhrase <> '' GROUP BY SearchPhrase ORDER BY c DESC LIMIT 10;
Time: 27041,816 ms (00:27,042)
Time: 27377,888 ms (00:27,378)
Time: 27712,654 ms (00:27,713)
3
SELECT SearchPhrase, COUNT(DISTINCT UserID) AS u FROM hits WHERE SearchPhrase <> '' GROUP BY SearchPhrase ORDER BY u DESC LIMIT 10;
Time: 25028,066 ms (00:25,028)
Time: 25501,758 ms (00:25,502)
Time: 25481,128 ms (00:25,481)
3
SELECT SearchEngineID, SearchPhrase, COUNT(*) AS c FROM hits WHERE SearchPhrase <> '' GROUP BY SearchEngineID, SearchPhrase ORDER BY c DESC LIMIT 10;
Time: 30589,232 ms (00:30,589)
Time: 30913,761 ms (00:30,914)
Time: 31084,063 ms (00:31,084)
3
SELECT UserID, COUNT(*) FROM hits GROUP BY UserID ORDER BY COUNT(*) DESC LIMIT 10;
Time: 50583,807 ms (00:50,584)
Time: 50055,109 ms (00:50,055)
Time: 50773,918 ms (00:50,774)
3
SELECT UserID, SearchPhrase, COUNT(*) FROM hits GROUP BY UserID, SearchPhrase ORDER BY COUNT(*) DESC LIMIT 10;
Time: 49792,403 ms (00:49,792)
Time: 49999,407 ms (00:49,999)
Time: 50561,455 ms (00:50,561)
3
SELECT UserID, SearchPhrase, COUNT(*) FROM hits GROUP BY UserID, SearchPhrase LIMIT 10;
Time: 39494,417 ms (00:39,494)
Time: 40946,123 ms (00:40,946)
Time: 39974,006 ms (00:39,974)
3
SELECT UserID, extract(minute FROM EventTime) AS m, SearchPhrase, COUNT(*) FROM hits GROUP BY UserID, m, SearchPhrase ORDER BY COUNT(*) DESC LIMIT 10;
Time: 58471,701 ms (00:58,472)
Time: 58863,934 ms (00:58,864)
Time: 58762,880 ms (00:58,763)
3
SELECT UserID FROM hits WHERE UserID = 435090932899640449;
Time: 13201,426 ms (00:13,201)
Time: 13566,324 ms (00:13,566)
Time: 13457,805 ms (00:13,458)
3
SELECT COUNT(*) FROM hits WHERE URL LIKE '%google%';
Time: 15539,853 ms (00:15,540)
Time: 15819,455 ms (00:15,819)
Time: 15691,132 ms (00:15,691)
3
SELECT SearchPhrase, MIN(URL), COUNT(*) AS c FROM hits WHERE URL LIKE '%google%' AND SearchPhrase <> '' GROUP BY SearchPhrase ORDER BY c DESC LIMIT 10;
Time: 17373,481 ms (00:17,373)
Time: 17909,029 ms (00:17,909)
Time: 17971,746 ms (00:17,972)
3
SELECT SearchPhrase, MIN(URL), MIN(Title), COUNT(*) AS c, COUNT(DISTINCT UserID) FROM hits WHERE Title LIKE '%Google%' AND URL NOT LIKE '%.google.%' AND SearchPhrase <> '' GROUP BY SearchPhrase ORDER BY c DESC LIMIT 10;
Time: 17356,677 ms (00:17,357)
Time: 17934,457 ms (00:17,934)
Time: 18107,978 ms (00:18,108)
3
SELECT * FROM hits WHERE URL LIKE '%google%' ORDER BY EventTime LIMIT 10;
Time: 15189,144 ms (00:15,189)
Time: 15910,373 ms (00:15,910)
Time: 15860,798 ms (00:15,861)
3
SELECT SearchPhrase FROM hits WHERE SearchPhrase <> '' ORDER BY EventTime LIMIT 10;
Time: 15271,197 ms (00:15,271)
Time: 15893,442 ms (00:15,893)
Time: 15588,961 ms (00:15,589)
3
SELECT SearchPhrase FROM hits WHERE SearchPhrase <> '' ORDER BY SearchPhrase LIMIT 10;
Time: 15315,928 ms (00:15,316)
Time: 15864,849 ms (00:15,865)
Time: 15697,214 ms (00:15,697)
3
SELECT SearchPhrase FROM hits WHERE SearchPhrase <> '' ORDER BY EventTime, SearchPhrase LIMIT 10;
Time: 15230,477 ms (00:15,230)
Time: 15801,559 ms (00:15,802)
Time: 15689,589 ms (00:15,690)
3
SELECT CounterID, AVG(length(URL)) AS l, COUNT(*) AS c FROM hits WHERE URL <> '' GROUP BY CounterID HAVING COUNT(*) > 100000 ORDER BY l DESC LIMIT 25;
Time: 18499,124 ms (00:18,499)
Time: 19145,595 ms (00:19,146)
Time: 18814,859 ms (00:18,815)
3
SELECT REGEXP_REPLACE(Referer, '^https?://(?:www.)?([^/]+)/.*$', '1') AS k, AVG(length(Referer)) AS l, COUNT(*) AS c, MIN(Referer) FROM hits WHERE Referer <> '' GROUP BY k HAVING COUNT(*) > 100000 ORDER BY l DESC LIMIT 25;
Time: 71306,407 ms (01:11,306)
Time: 71630,846 ms (01:11,631)
Time: 71999,890 ms (01:12,000)
3
SELECT SUM(ResolutionWidth), SUM(ResolutionWidth + 1), SUM(ResolutionWidth + 2), SUM(ResolutionWidth + 3), SUM(ResolutionWidth + 4), SUM(ResolutionWidth + 5), SUM(ResolutionWidth + 6), SUM(ResolutionWidth + 7), SUM(ResolutionWidth + 8), SUM(ResolutionWidth + 9), SUM(ResolutionWidth + 10), SUM(ResolutionWidth + 11), SUM(ResolutionWidth + 12), SUM(ResolutionWidth + 13), SUM(ResolutionWidth + 14), SUM(ResolutionWidth + 15), SUM(ResolutionWidth + 16), SUM(ResolutionWidth + 17), SUM(ResolutionWidth + 18), SUM(ResolutionWidth + 19), SUM(ResolutionWidth + 20), SUM(ResolutionWidth + 21), SUM(ResolutionWidth + 22), SUM(ResolutionWidth + 23), SUM(ResolutionWidth + 24), SUM(ResolutionWidth + 25), SUM(ResolutionWidth + 26), SUM(ResolutionWidth + 27), SUM(ResolutionWidth + 28), SUM(ResolutionWidth + 29), SUM(ResolutionWidth + 30), SUM(ResolutionWidth + 31), SUM(ResolutionWidth + 32), SUM(ResolutionWidth + 33), SUM(ResolutionWidth + 34), SUM(ResolutionWidth + 35), SUM(ResolutionWidth + 36), SUM(ResolutionWidth + 37), SUM(ResolutionWidth + 38), SUM(ResolutionWidth + 39), SUM(ResolutionWidth + 40), SUM(ResolutionWidth + 41), SUM(ResolutionWidth + 42), SUM(ResolutionWidth + 43), SUM(ResolutionWidth + 44), SUM(ResolutionWidth + 45), SUM(ResolutionWidth + 46), SUM(ResolutionWidth + 47), SUM(ResolutionWidth + 48), SUM(ResolutionWidth + 49), SUM(ResolutionWidth + 50), SUM(ResolutionWidth + 51), SUM(ResolutionWidth + 52), SUM(ResolutionWidth + 53), SUM(ResolutionWidth + 54), SUM(ResolutionWidth + 55), SUM(ResolutionWidth + 56), SUM(ResolutionWidth + 57), SUM(ResolutionWidth + 58), SUM(ResolutionWidth + 59), SUM(ResolutionWidth + 60), SUM(ResolutionWidth + 61), SUM(ResolutionWidth + 62), SUM(ResolutionWidth + 63), SUM(ResolutionWidth + 64), SUM(ResolutionWidth + 65), SUM(ResolutionWidth + 66), SUM(ResolutionWidth + 67), SUM(ResolutionWidth + 68), SUM(ResolutionWidth + 69), SUM(ResolutionWidth + 70), SUM(ResolutionWidth + 71), SUM(ResolutionWidth + 72), SUM(ResolutionWidth + 73), SUM(ResolutionWidth + 74), SUM(ResolutionWidth + 75), SUM(ResolutionWidth + 76), SUM(ResolutionWidth + 77), SUM(ResolutionWidth + 78), SUM(ResolutionWidth + 79), SUM(ResolutionWidth + 80), SUM(ResolutionWidth + 81), SUM(ResolutionWidth + 82), SUM(ResolutionWidth + 83), SUM(ResolutionWidth + 84), SUM(ResolutionWidth + 85), SUM(ResolutionWidth + 86), SUM(ResolutionWidth + 87), SUM(ResolutionWidth + 88), SUM(ResolutionWidth + 89) FROM hits;
Time: 36252,969 ms (00:36,253)
Time: 36317,367 ms (00:36,317)
Time: 36504,270 ms (00:36,504)
3
SELECT SearchEngineID, ClientIP, COUNT(*) AS c, SUM(IsRefresh), AVG(ResolutionWidth) FROM hits WHERE SearchPhrase <> '' GROUP BY SearchEngineID, ClientIP ORDER BY c DESC LIMIT 10;
Time: 21918,328 ms (00:21,918)
Time: 22863,693 ms (00:22,864)
Time: 22395,174 ms (00:22,395)
3
SELECT WatchID, ClientIP, COUNT(*) AS c, SUM(IsRefresh), AVG(ResolutionWidth) FROM hits WHERE SearchPhrase <> '' GROUP BY WatchID, ClientIP ORDER BY c DESC LIMIT 10;
Time: 22463,163 ms (00:22,463)
Time: 22820,475 ms (00:22,820)
Time: 22533,673 ms (00:22,534)
3
SELECT WatchID, ClientIP, COUNT(*) AS c, SUM(IsRefresh), AVG(ResolutionWidth) FROM hits GROUP BY WatchID, ClientIP ORDER BY c DESC LIMIT 10;
Time: 176243,507 ms (02:56,244)
Time: 175242,545 ms (02:55,243)
Time: 177691,352 ms (02:57,691)
3
SELECT URL, COUNT(*) AS c FROM hits GROUP BY URL ORDER BY c DESC LIMIT 10;
Time: 89020,000 ms (01:29,020)
Time: 89472,048 ms (01:29,472)
Time: 89393,561 ms (01:29,394)
3
SELECT 1, URL, COUNT(*) AS c FROM hits GROUP BY 1, URL ORDER BY c DESC LIMIT 10;
Time: 90514,518 ms (01:30,515)
Time: 89464,720 ms (01:29,465)
Time: 89692,554 ms (01:29,693)
3
SELECT ClientIP, ClientIP - 1, ClientIP - 2, ClientIP - 3, COUNT(*) AS c FROM hits GROUP BY ClientIP, ClientIP - 1, ClientIP - 2, ClientIP - 3 ORDER BY c DESC LIMIT 10;
Time: 26606,945 ms (00:26,607)
Time: 28087,672 ms (00:28,088)
Time: 27841,070 ms (00:27,841)
3
SELECT URL, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND DontCountHits = 0 AND IsRefresh = 0 AND URL <> '' GROUP BY URL ORDER BY PageViews DESC LIMIT 10;
Time: 17349,534 ms (00:17,350)
Time: 17787,679 ms (00:17,788)
Time: 17988,118 ms (00:17,988)
3
SELECT Title, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND DontCountHits = 0 AND IsRefresh = 0 AND Title <> '' GROUP BY Title ORDER BY PageViews DESC LIMIT 10;
Time: 17075,309 ms (00:17,075)
Time: 17517,364 ms (00:17,517)
Time: 17489,677 ms (00:17,490)
3
SELECT URL, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND IsRefresh = 0 AND IsLink <> 0 AND IsDownload = 0 GROUP BY URL ORDER BY PageViews DESC LIMIT 10 OFFSET 1000;
Time: 15944,587 ms (00:15,945)
Time: 16464,463 ms (00:16,464)
Time: 16441,888 ms (00:16,442)
3
SELECT TraficSourceID, SearchEngineID, AdvEngineID, CASE WHEN (SearchEngineID = 0 AND AdvEngineID = 0) THEN Referer ELSE '' END AS Src, URL AS Dst, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND IsRefresh = 0 GROUP BY TraficSourceID, SearchEngineID, AdvEngineID, Src, Dst ORDER BY PageViews DESC LIMIT 10 OFFSET 1000;
Time: 14605,393 ms (00:14,605)
Time: 14929,000 ms (00:14,929)
Time: 14801,557 ms (00:14,802)
3
SELECT URLHash, EventDate, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND IsRefresh = 0 AND TraficSourceID IN (-1, 6) AND RefererHash = 3594120000172545465 GROUP BY URLHash, EventDate ORDER BY PageViews DESC LIMIT 10 OFFSET 100;
Time: 20570,407 ms (00:20,570)
Time: 21115,031 ms (00:21,115)
Time: 21042,148 ms (00:21,042)
3
SELECT WindowClientWidth, WindowClientHeight, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND IsRefresh = 0 AND DontCountHits = 0 AND URLHash = 2868770270353813622 GROUP BY WindowClientWidth, WindowClientHeight ORDER BY PageViews DESC LIMIT 10 OFFSET 10000;
Time: 20581,186 ms (00:20,581)
Time: 20882,095 ms (00:20,882)
Time: 20888,331 ms (00:20,888)
3
SELECT DATE_TRUNC('minute', EventTime) AS M, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-14' AND EventDate <= '2013-07-15' AND IsRefresh = 0 AND DontCountHits = 0 GROUP BY DATE_TRUNC('minute', EventTime) ORDER BY DATE_TRUNC('minute', EventTime) LIMIT 10 OFFSET 1000;
Time: 16619,404 ms (00:16,619)
Time: 17223,469 ms (00:17,223)
Time: 16872,552 ms (00:16,873)


  ```
</details>

<details>
  <summary>Release with LTO and -march=native</summary>
  
  ```
  3
SELECT COUNT(*) FROM hits;
Time: 12932,540 ms (00:12,933)
Time: 13131,253 ms (00:13,131)
Time: 13026,592 ms (00:13,027)
3
SELECT COUNT(*) FROM hits WHERE AdvEngineID <> 0;
Time: 14431,464 ms (00:14,431)
Time: 15248,575 ms (00:15,249)
Time: 14962,596 ms (00:14,963)
3
SELECT SUM(AdvEngineID), COUNT(*), AVG(ResolutionWidth) FROM hits;
Time: 15051,184 ms (00:15,051)
Time: 15603,215 ms (00:15,603)
Time: 15499,951 ms (00:15,500)
3
SELECT AVG(UserID) FROM hits;
Time: 13194,350 ms (00:13,194)
Time: 13500,205 ms (00:13,500)
Time: 13503,151 ms (00:13,503)
3
SELECT COUNT(DISTINCT UserID) FROM hits;
Time: 42717,728 ms (00:42,718)
Time: 43526,385 ms (00:43,526)
Time: 43567,363 ms (00:43,567)
3
SELECT COUNT(DISTINCT SearchPhrase) FROM hits;
Time: 72858,453 ms (01:12,858)
Time: 73406,719 ms (01:13,407)
Time: 73317,471 ms (01:13,317)
3
SELECT MIN(EventDate), MAX(EventDate) FROM hits;
Time: 13236,641 ms (00:13,237)
Time: 13621,025 ms (00:13,621)
Time: 13410,509 ms (00:13,411)
3
SELECT AdvEngineID, COUNT(*) FROM hits WHERE AdvEngineID <> 0 GROUP BY AdvEngineID ORDER BY COUNT(*) DESC;
Time: 14576,374 ms (00:14,576)
Time: 15186,620 ms (00:15,187)
Time: 15148,302 ms (00:15,148)
3
SELECT RegionID, COUNT(DISTINCT UserID) AS u FROM hits GROUP BY RegionID ORDER BY u DESC LIMIT 10;
Time: 32444,346 ms (00:32,444)
Time: 33225,815 ms (00:33,226)
Time: 33335,646 ms (00:33,336)
3
SELECT RegionID, SUM(AdvEngineID), COUNT(*) AS c, AVG(ResolutionWidth), COUNT(DISTINCT UserID) FROM hits GROUP BY RegionID ORDER BY c DESC LIMIT 10;
Time: 37389,268 ms (00:37,389)
Time: 37566,728 ms (00:37,567)
Time: 37464,756 ms (00:37,465)
3
SELECT MobilePhoneModel, COUNT(DISTINCT UserID) AS u FROM hits WHERE MobilePhoneModel <> '' GROUP BY MobilePhoneModel ORDER BY u DESC LIMIT 10;
Time: 16257,351 ms (00:16,257)
Time: 16828,935 ms (00:16,829)
Time: 16598,099 ms (00:16,598)
3
SELECT MobilePhone, MobilePhoneModel, COUNT(DISTINCT UserID) AS u FROM hits WHERE MobilePhoneModel <> '' GROUP BY MobilePhone, MobilePhoneModel ORDER BY u DESC LIMIT 10;
Time: 16828,651 ms (00:16,829)
Time: 17482,272 ms (00:17,482)
Time: 17247,338 ms (00:17,247)
3
SELECT SearchPhrase, COUNT(*) AS c FROM hits WHERE SearchPhrase <> '' GROUP BY SearchPhrase ORDER BY c DESC LIMIT 10;
Time: 26164,833 ms (00:26,165)
Time: 27171,224 ms (00:27,171)
Time: 27030,558 ms (00:27,031)
3
SELECT SearchPhrase, COUNT(DISTINCT UserID) AS u FROM hits WHERE SearchPhrase <> '' GROUP BY SearchPhrase ORDER BY u DESC LIMIT 10;
Time: 25024,126 ms (00:25,024)
Time: 25592,813 ms (00:25,593)
Time: 25277,955 ms (00:25,278)
3
SELECT SearchEngineID, SearchPhrase, COUNT(*) AS c FROM hits WHERE SearchPhrase <> '' GROUP BY SearchEngineID, SearchPhrase ORDER BY c DESC LIMIT 10;
Time: 29804,871 ms (00:29,805)
Time: 30587,631 ms (00:30,588)
Time: 29873,848 ms (00:29,874)
3
SELECT UserID, COUNT(*) FROM hits GROUP BY UserID ORDER BY COUNT(*) DESC LIMIT 10;
Time: 48527,354 ms (00:48,527)
Time: 48961,592 ms (00:48,962)
Time: 49348,450 ms (00:49,348)
3
SELECT UserID, SearchPhrase, COUNT(*) FROM hits GROUP BY UserID, SearchPhrase ORDER BY COUNT(*) DESC LIMIT 10;
Time: 48822,209 ms (00:48,822)
Time: 50457,854 ms (00:50,458)
Time: 49519,229 ms (00:49,519)
3
SELECT UserID, SearchPhrase, COUNT(*) FROM hits GROUP BY UserID, SearchPhrase LIMIT 10;
Time: 38586,266 ms (00:38,586)
Time: 39463,272 ms (00:39,463)
Time: 38972,762 ms (00:38,973)
3
SELECT UserID, extract(minute FROM EventTime) AS m, SearchPhrase, COUNT(*) FROM hits GROUP BY UserID, m, SearchPhrase ORDER BY COUNT(*) DESC LIMIT 10;
Time: 58801,156 ms (00:58,801)
Time: 59332,211 ms (00:59,332)
Time: 60111,880 ms (01:00,112)
3
SELECT UserID FROM hits WHERE UserID = 435090932899640449;
Time: 13218,417 ms (00:13,218)
Time: 13583,591 ms (00:13,584)
Time: 13390,686 ms (00:13,391)
3
SELECT COUNT(*) FROM hits WHERE URL LIKE '%google%';
Time: 15496,234 ms (00:15,496)
Time: 15746,398 ms (00:15,746)
Time: 15705,522 ms (00:15,706)
3
SELECT SearchPhrase, MIN(URL), COUNT(*) AS c FROM hits WHERE URL LIKE '%google%' AND SearchPhrase <> '' GROUP BY SearchPhrase ORDER BY c DESC LIMIT 10;
Time: 17191,473 ms (00:17,191)
Time: 17880,713 ms (00:17,881)
Time: 17506,472 ms (00:17,506)
3
SELECT SearchPhrase, MIN(URL), MIN(Title), COUNT(*) AS c, COUNT(DISTINCT UserID) FROM hits WHERE Title LIKE '%Google%' AND URL NOT LIKE '%.google.%' AND SearchPhrase <> '' GROUP BY SearchPhrase ORDER BY c DESC LIMIT 10;
Time: 17478,770 ms (00:17,479)
Time: 18118,918 ms (00:18,119)
Time: 17688,405 ms (00:17,688)
3
SELECT * FROM hits WHERE URL LIKE '%google%' ORDER BY EventTime LIMIT 10;
Time: 15364,385 ms (00:15,364)
Time: 15894,221 ms (00:15,894)
Time: 15841,870 ms (00:15,842)
3
SELECT SearchPhrase FROM hits WHERE SearchPhrase <> '' ORDER BY EventTime LIMIT 10;
Time: 15234,704 ms (00:15,235)
Time: 15675,427 ms (00:15,675)
Time: 15431,280 ms (00:15,431)
3
SELECT SearchPhrase FROM hits WHERE SearchPhrase <> '' ORDER BY SearchPhrase LIMIT 10;
Time: 15097,598 ms (00:15,098)
Time: 15580,424 ms (00:15,580)
Time: 15583,833 ms (00:15,584)
3
SELECT SearchPhrase FROM hits WHERE SearchPhrase <> '' ORDER BY EventTime, SearchPhrase LIMIT 10;
Time: 15263,878 ms (00:15,264)
Time: 15815,791 ms (00:15,816)
Time: 15808,763 ms (00:15,809)
3
SELECT CounterID, AVG(length(URL)) AS l, COUNT(*) AS c FROM hits WHERE URL <> '' GROUP BY CounterID HAVING COUNT(*) > 100000 ORDER BY l DESC LIMIT 25;
Time: 18380,757 ms (00:18,381)
Time: 18784,727 ms (00:18,785)
Time: 19070,988 ms (00:19,071)
3
SELECT REGEXP_REPLACE(Referer, '^https?://(?:www.)?([^/]+)/.*$', '1') AS k, AVG(length(Referer)) AS l, COUNT(*) AS c, MIN(Referer) FROM hits WHERE Referer <> '' GROUP BY k HAVING COUNT(*) > 100000 ORDER BY l DESC LIMIT 25;
Time: 77306,786 ms (01:17,307)
Time: 77946,043 ms (01:17,946)
Time: 77574,454 ms (01:17,574)
3
SELECT SUM(ResolutionWidth), SUM(ResolutionWidth + 1), SUM(ResolutionWidth + 2), SUM(ResolutionWidth + 3), SUM(ResolutionWidth + 4), SUM(ResolutionWidth + 5), SUM(ResolutionWidth + 6), SUM(ResolutionWidth + 7), SUM(ResolutionWidth + 8), SUM(ResolutionWidth + 9), SUM(ResolutionWidth + 10), SUM(ResolutionWidth + 11), SUM(ResolutionWidth + 12), SUM(ResolutionWidth + 13), SUM(ResolutionWidth + 14), SUM(ResolutionWidth + 15), SUM(ResolutionWidth + 16), SUM(ResolutionWidth + 17), SUM(ResolutionWidth + 18), SUM(ResolutionWidth + 19), SUM(ResolutionWidth + 20), SUM(ResolutionWidth + 21), SUM(ResolutionWidth + 22), SUM(ResolutionWidth + 23), SUM(ResolutionWidth + 24), SUM(ResolutionWidth + 25), SUM(ResolutionWidth + 26), SUM(ResolutionWidth + 27), SUM(ResolutionWidth + 28), SUM(ResolutionWidth + 29), SUM(ResolutionWidth + 30), SUM(ResolutionWidth + 31), SUM(ResolutionWidth + 32), SUM(ResolutionWidth + 33), SUM(ResolutionWidth + 34), SUM(ResolutionWidth + 35), SUM(ResolutionWidth + 36), SUM(ResolutionWidth + 37), SUM(ResolutionWidth + 38), SUM(ResolutionWidth + 39), SUM(ResolutionWidth + 40), SUM(ResolutionWidth + 41), SUM(ResolutionWidth + 42), SUM(ResolutionWidth + 43), SUM(ResolutionWidth + 44), SUM(ResolutionWidth + 45), SUM(ResolutionWidth + 46), SUM(ResolutionWidth + 47), SUM(ResolutionWidth + 48), SUM(ResolutionWidth + 49), SUM(ResolutionWidth + 50), SUM(ResolutionWidth + 51), SUM(ResolutionWidth + 52), SUM(ResolutionWidth + 53), SUM(ResolutionWidth + 54), SUM(ResolutionWidth + 55), SUM(ResolutionWidth + 56), SUM(ResolutionWidth + 57), SUM(ResolutionWidth + 58), SUM(ResolutionWidth + 59), SUM(ResolutionWidth + 60), SUM(ResolutionWidth + 61), SUM(ResolutionWidth + 62), SUM(ResolutionWidth + 63), SUM(ResolutionWidth + 64), SUM(ResolutionWidth + 65), SUM(ResolutionWidth + 66), SUM(ResolutionWidth + 67), SUM(ResolutionWidth + 68), SUM(ResolutionWidth + 69), SUM(ResolutionWidth + 70), SUM(ResolutionWidth + 71), SUM(ResolutionWidth + 72), SUM(ResolutionWidth + 73), SUM(ResolutionWidth + 74), SUM(ResolutionWidth + 75), SUM(ResolutionWidth + 76), SUM(ResolutionWidth + 77), SUM(ResolutionWidth + 78), SUM(ResolutionWidth + 79), SUM(ResolutionWidth + 80), SUM(ResolutionWidth + 81), SUM(ResolutionWidth + 82), SUM(ResolutionWidth + 83), SUM(ResolutionWidth + 84), SUM(ResolutionWidth + 85), SUM(ResolutionWidth + 86), SUM(ResolutionWidth + 87), SUM(ResolutionWidth + 88), SUM(ResolutionWidth + 89) FROM hits;
Time: 35679,394 ms (00:35,679)
Time: 36268,086 ms (00:36,268)
Time: 35655,459 ms (00:35,655)
3
SELECT SearchEngineID, ClientIP, COUNT(*) AS c, SUM(IsRefresh), AVG(ResolutionWidth) FROM hits WHERE SearchPhrase <> '' GROUP BY SearchEngineID, ClientIP ORDER BY c DESC LIMIT 10;
Time: 22104,133 ms (00:22,104)
Time: 22312,264 ms (00:22,312)
Time: 22751,519 ms (00:22,752)
3
SELECT WatchID, ClientIP, COUNT(*) AS c, SUM(IsRefresh), AVG(ResolutionWidth) FROM hits WHERE SearchPhrase <> '' GROUP BY WatchID, ClientIP ORDER BY c DESC LIMIT 10;
Time: 22478,821 ms (00:22,479)
Time: 22993,115 ms (00:22,993)
Time: 23225,900 ms (00:23,226)
3
SELECT WatchID, ClientIP, COUNT(*) AS c, SUM(IsRefresh), AVG(ResolutionWidth) FROM hits GROUP BY WatchID, ClientIP ORDER BY c DESC LIMIT 10;
Time: 174015,041 ms (02:54,015)
Time: 171403,140 ms (02:51,403)
Time: 175053,525 ms (02:55,054)
3
SELECT URL, COUNT(*) AS c FROM hits GROUP BY URL ORDER BY c DESC LIMIT 10;
Time: 87948,219 ms (01:27,948)
Time: 89009,224 ms (01:29,009)
Time: 87871,923 ms (01:27,872)
3
SELECT 1, URL, COUNT(*) AS c FROM hits GROUP BY 1, URL ORDER BY c DESC LIMIT 10;
Time: 88495,053 ms (01:28,495)
Time: 88900,536 ms (01:28,901)
Time: 88712,140 ms (01:28,712)
3
SELECT ClientIP, ClientIP - 1, ClientIP - 2, ClientIP - 3, COUNT(*) AS c FROM hits GROUP BY ClientIP, ClientIP - 1, ClientIP - 2, ClientIP - 3 ORDER BY c DESC LIMIT 10;
Time: 28116,258 ms (00:28,116)
Time: 28420,508 ms (00:28,421)
Time: 27991,247 ms (00:27,991)
3
SELECT URL, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND DontCountHits = 0 AND IsRefresh = 0 AND URL <> '' GROUP BY URL ORDER BY PageViews DESC LIMIT 10;
Time: 17131,121 ms (00:17,131)
Time: 17438,558 ms (00:17,439)
Time: 17552,961 ms (00:17,553)
3
SELECT Title, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND DontCountHits = 0 AND IsRefresh = 0 AND Title <> '' GROUP BY Title ORDER BY PageViews DESC LIMIT 10;
Time: 16962,887 ms (00:16,963)
Time: 17339,046 ms (00:17,339)
Time: 17331,073 ms (00:17,331)
3
SELECT URL, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND IsRefresh = 0 AND IsLink <> 0 AND IsDownload = 0 GROUP BY URL ORDER BY PageViews DESC LIMIT 10 OFFSET 1000;
Time: 15780,379 ms (00:15,780)
Time: 16365,616 ms (00:16,366)
Time: 16189,022 ms (00:16,189)
3
SELECT TraficSourceID, SearchEngineID, AdvEngineID, CASE WHEN (SearchEngineID = 0 AND AdvEngineID = 0) THEN Referer ELSE '' END AS Src, URL AS Dst, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND IsRefresh = 0 GROUP BY TraficSourceID, SearchEngineID, AdvEngineID, Src, Dst ORDER BY PageViews DESC LIMIT 10 OFFSET 1000;
Time: 14469,740 ms (00:14,470)
Time: 14838,300 ms (00:14,838)
Time: 15005,736 ms (00:15,006)
3
SELECT URLHash, EventDate, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND IsRefresh = 0 AND TraficSourceID IN (-1, 6) AND RefererHash = 3594120000172545465 GROUP BY URLHash, EventDate ORDER BY PageViews DESC LIMIT 10 OFFSET 100;
Time: 20373,363 ms (00:20,373)
Time: 20720,433 ms (00:20,720)
Time: 20953,076 ms (00:20,953)
3
SELECT WindowClientWidth, WindowClientHeight, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND IsRefresh = 0 AND DontCountHits = 0 AND URLHash = 2868770270353813622 GROUP BY WindowClientWidth, WindowClientHeight ORDER BY PageViews DESC LIMIT 10 OFFSET 10000;
Time: 19946,454 ms (00:19,946)
Time: 20454,525 ms (00:20,455)
Time: 20353,164 ms (00:20,353)
3
SELECT DATE_TRUNC('minute', EventTime) AS M, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-14' AND EventDate <= '2013-07-15' AND IsRefresh = 0 AND DontCountHits = 0 GROUP BY DATE_TRUNC('minute', EventTime) ORDER BY DATE_TRUNC('minute', EventTime) LIMIT 10 OFFSET 1000;
Time: 16302,125 ms (00:16,302)
Time: 16778,642 ms (00:16,779)
Time: 16932,949 ms (00:16,933)

  ```
</details>

<details>
  <summary>Release with LTO and PGO</summary>
  
  ```
  3
SELECT COUNT(*) FROM hits;
Time: 12849,937 ms (00:12,850)
Time: 12968,460 ms (00:12,968)
Time: 12890,595 ms (00:12,891)
3
SELECT COUNT(*) FROM hits WHERE AdvEngineID <> 0;
Time: 13861,661 ms (00:13,862)
Time: 14418,403 ms (00:14,418)
Time: 14411,771 ms (00:14,412)
3
SELECT SUM(AdvEngineID), COUNT(*), AVG(ResolutionWidth) FROM hits;
Time: 14248,633 ms (00:14,249)
Time: 14541,057 ms (00:14,541)
Time: 14543,539 ms (00:14,544)
3
SELECT AVG(UserID) FROM hits;
Time: 13042,075 ms (00:13,042)
Time: 13390,566 ms (00:13,391)
Time: 13228,156 ms (00:13,228)
3
SELECT COUNT(DISTINCT UserID) FROM hits;
Time: 39967,719 ms (00:39,968)
Time: 39246,143 ms (00:39,246)
Time: 40363,201 ms (00:40,363)
3
SELECT COUNT(DISTINCT SearchPhrase) FROM hits;
Time: 63770,471 ms (01:03,770)
Time: 63873,872 ms (01:03,874)
Time: 63466,881 ms (01:03,467)
3
SELECT MIN(EventDate), MAX(EventDate) FROM hits;
Time: 13089,015 ms (00:13,089)
Time: 13224,935 ms (00:13,225)
Time: 13219,853 ms (00:13,220)
3
SELECT AdvEngineID, COUNT(*) FROM hits WHERE AdvEngineID <> 0 GROUP BY AdvEngineID ORDER BY COUNT(*) DESC;
Time: 14007,883 ms (00:14,008)
Time: 14514,005 ms (00:14,514)
Time: 14255,166 ms (00:14,255)
3
SELECT RegionID, COUNT(DISTINCT UserID) AS u FROM hits GROUP BY RegionID ORDER BY u DESC LIMIT 10;
Time: 29560,894 ms (00:29,561)
Time: 30171,434 ms (00:30,171)
Time: 30414,179 ms (00:30,414)
3
SELECT RegionID, SUM(AdvEngineID), COUNT(*) AS c, AVG(ResolutionWidth), COUNT(DISTINCT UserID) FROM hits GROUP BY RegionID ORDER BY c DESC LIMIT 10;
Time: 33539,364 ms (00:33,539)
Time: 34203,529 ms (00:34,204)
Time: 34576,398 ms (00:34,576)
3
SELECT MobilePhoneModel, COUNT(DISTINCT UserID) AS u FROM hits WHERE MobilePhoneModel <> '' GROUP BY MobilePhoneModel ORDER BY u DESC LIMIT 10;
Time: 14910,824 ms (00:14,911)
Time: 15539,577 ms (00:15,540)
Time: 15475,645 ms (00:15,476)
3
SELECT MobilePhone, MobilePhoneModel, COUNT(DISTINCT UserID) AS u FROM hits WHERE MobilePhoneModel <> '' GROUP BY MobilePhone, MobilePhoneModel ORDER BY u DESC LIMIT 10;
Time: 15529,283 ms (00:15,529)
Time: 15995,029 ms (00:15,995)
Time: 15796,465 ms (00:15,796)
3
SELECT SearchPhrase, COUNT(*) AS c FROM hits WHERE SearchPhrase <> '' GROUP BY SearchPhrase ORDER BY c DESC LIMIT 10;
Time: 24208,248 ms (00:24,208)
Time: 25155,116 ms (00:25,155)
Time: 24776,029 ms (00:24,776)
3
SELECT SearchPhrase, COUNT(DISTINCT UserID) AS u FROM hits WHERE SearchPhrase <> '' GROUP BY SearchPhrase ORDER BY u DESC LIMIT 10;
Time: 22530,711 ms (00:22,531)
Time: 22845,442 ms (00:22,845)
Time: 23196,489 ms (00:23,196)
3
SELECT SearchEngineID, SearchPhrase, COUNT(*) AS c FROM hits WHERE SearchPhrase <> '' GROUP BY SearchEngineID, SearchPhrase ORDER BY c DESC LIMIT 10;
Time: 27247,566 ms (00:27,248)
Time: 27464,872 ms (00:27,465)
Time: 27225,327 ms (00:27,225)
3
SELECT UserID, COUNT(*) FROM hits GROUP BY UserID ORDER BY COUNT(*) DESC LIMIT 10;
Time: 44448,925 ms (00:44,449)
Time: 44345,792 ms (00:44,346)
Time: 44423,715 ms (00:44,424)
3
SELECT UserID, SearchPhrase, COUNT(*) FROM hits GROUP BY UserID, SearchPhrase ORDER BY COUNT(*) DESC LIMIT 10;
Time: 44088,001 ms (00:44,088)
Time: 44758,910 ms (00:44,759)
Time: 44971,292 ms (00:44,971)
3
SELECT UserID, SearchPhrase, COUNT(*) FROM hits GROUP BY UserID, SearchPhrase LIMIT 10;
Time: 34578,618 ms (00:34,579)
Time: 35616,509 ms (00:35,617)
Time: 35530,003 ms (00:35,530)
3
SELECT UserID, extract(minute FROM EventTime) AS m, SearchPhrase, COUNT(*) FROM hits GROUP BY UserID, m, SearchPhrase ORDER BY COUNT(*) DESC LIMIT 10;
Time: 51120,124 ms (00:51,120)
Time: 52011,253 ms (00:52,011)
Time: 52473,116 ms (00:52,473)
3
SELECT UserID FROM hits WHERE UserID = 435090932899640449;
Time: 13041,037 ms (00:13,041)
Time: 13313,663 ms (00:13,314)
Time: 13103,621 ms (00:13,104)
3
SELECT COUNT(*) FROM hits WHERE URL LIKE '%google%';
Time: 14968,166 ms (00:14,968)
Time: 15189,666 ms (00:15,190)
Time: 15117,787 ms (00:15,118)
3
SELECT SearchPhrase, MIN(URL), COUNT(*) AS c FROM hits WHERE URL LIKE '%google%' AND SearchPhrase <> '' GROUP BY SearchPhrase ORDER BY c DESC LIMIT 10;
Time: 16077,852 ms (00:16,078)
Time: 16765,811 ms (00:16,766)
Time: 16529,276 ms (00:16,529)
3
SELECT SearchPhrase, MIN(URL), MIN(Title), COUNT(*) AS c, COUNT(DISTINCT UserID) FROM hits WHERE Title LIKE '%Google%' AND URL NOT LIKE '%.google.%' AND SearchPhrase <> '' GROUP BY SearchPhrase ORDER BY c DESC LIMIT 10;
Time: 16564,222 ms (00:16,564)
Time: 16986,238 ms (00:16,986)
Time: 16811,651 ms (00:16,812)
3
SELECT * FROM hits WHERE URL LIKE '%google%' ORDER BY EventTime LIMIT 10;
Time: 14958,168 ms (00:14,958)
Time: 15286,617 ms (00:15,287)
Time: 15251,485 ms (00:15,251)
3
SELECT SearchPhrase FROM hits WHERE SearchPhrase <> '' ORDER BY EventTime LIMIT 10;
Time: 14142,643 ms (00:14,143)
Time: 14840,880 ms (00:14,841)
Time: 16103,953 ms (00:16,104)
3
SELECT SearchPhrase FROM hits WHERE SearchPhrase <> '' ORDER BY SearchPhrase LIMIT 10;
Time: 16049,114 ms (00:16,049)
Time: 16091,315 ms (00:16,091)
Time: 16157,036 ms (00:16,157)
3
SELECT SearchPhrase FROM hits WHERE SearchPhrase <> '' ORDER BY EventTime, SearchPhrase LIMIT 10;
Time: 14263,352 ms (00:14,263)
Time: 14837,307 ms (00:14,837)
Time: 14576,825 ms (00:14,577)
3
SELECT CounterID, AVG(length(URL)) AS l, COUNT(*) AS c FROM hits WHERE URL <> '' GROUP BY CounterID HAVING COUNT(*) > 100000 ORDER BY l DESC LIMIT 25;
Time: 17397,768 ms (00:17,398)
Time: 17983,547 ms (00:17,984)
Time: 18010,883 ms (00:18,011)
3
SELECT REGEXP_REPLACE(Referer, '^https?://(?:www.)?([^/]+)/.*$', '1') AS k, AVG(length(Referer)) AS l, COUNT(*) AS c, MIN(Referer) FROM hits WHERE Referer <> '' GROUP BY k HAVING COUNT(*) > 100000 ORDER BY l DESC LIMIT 25;
Time: 65773,916 ms (01:05,774)
Time: 66212,221 ms (01:06,212)
Time: 66431,467 ms (01:06,431)
3
SELECT SUM(ResolutionWidth), SUM(ResolutionWidth + 1), SUM(ResolutionWidth + 2), SUM(ResolutionWidth + 3), SUM(ResolutionWidth + 4), SUM(ResolutionWidth + 5), SUM(ResolutionWidth + 6), SUM(ResolutionWidth + 7), SUM(ResolutionWidth + 8), SUM(ResolutionWidth + 9), SUM(ResolutionWidth + 10), SUM(ResolutionWidth + 11), SUM(ResolutionWidth + 12), SUM(ResolutionWidth + 13), SUM(ResolutionWidth + 14), SUM(ResolutionWidth + 15), SUM(ResolutionWidth + 16), SUM(ResolutionWidth + 17), SUM(ResolutionWidth + 18), SUM(ResolutionWidth + 19), SUM(ResolutionWidth + 20), SUM(ResolutionWidth + 21), SUM(ResolutionWidth + 22), SUM(ResolutionWidth + 23), SUM(ResolutionWidth + 24), SUM(ResolutionWidth + 25), SUM(ResolutionWidth + 26), SUM(ResolutionWidth + 27), SUM(ResolutionWidth + 28), SUM(ResolutionWidth + 29), SUM(ResolutionWidth + 30), SUM(ResolutionWidth + 31), SUM(ResolutionWidth + 32), SUM(ResolutionWidth + 33), SUM(ResolutionWidth + 34), SUM(ResolutionWidth + 35), SUM(ResolutionWidth + 36), SUM(ResolutionWidth + 37), SUM(ResolutionWidth + 38), SUM(ResolutionWidth + 39), SUM(ResolutionWidth + 40), SUM(ResolutionWidth + 41), SUM(ResolutionWidth + 42), SUM(ResolutionWidth + 43), SUM(ResolutionWidth + 44), SUM(ResolutionWidth + 45), SUM(ResolutionWidth + 46), SUM(ResolutionWidth + 47), SUM(ResolutionWidth + 48), SUM(ResolutionWidth + 49), SUM(ResolutionWidth + 50), SUM(ResolutionWidth + 51), SUM(ResolutionWidth + 52), SUM(ResolutionWidth + 53), SUM(ResolutionWidth + 54), SUM(ResolutionWidth + 55), SUM(ResolutionWidth + 56), SUM(ResolutionWidth + 57), SUM(ResolutionWidth + 58), SUM(ResolutionWidth + 59), SUM(ResolutionWidth + 60), SUM(ResolutionWidth + 61), SUM(ResolutionWidth + 62), SUM(ResolutionWidth + 63), SUM(ResolutionWidth + 64), SUM(ResolutionWidth + 65), SUM(ResolutionWidth + 66), SUM(ResolutionWidth + 67), SUM(ResolutionWidth + 68), SUM(ResolutionWidth + 69), SUM(ResolutionWidth + 70), SUM(ResolutionWidth + 71), SUM(ResolutionWidth + 72), SUM(ResolutionWidth + 73), SUM(ResolutionWidth + 74), SUM(ResolutionWidth + 75), SUM(ResolutionWidth + 76), SUM(ResolutionWidth + 77), SUM(ResolutionWidth + 78), SUM(ResolutionWidth + 79), SUM(ResolutionWidth + 80), SUM(ResolutionWidth + 81), SUM(ResolutionWidth + 82), SUM(ResolutionWidth + 83), SUM(ResolutionWidth + 84), SUM(ResolutionWidth + 85), SUM(ResolutionWidth + 86), SUM(ResolutionWidth + 87), SUM(ResolutionWidth + 88), SUM(ResolutionWidth + 89) FROM hits;
Time: 35557,873 ms (00:35,558)
Time: 36281,014 ms (00:36,281)
Time: 35931,948 ms (00:35,932)
3
SELECT SearchEngineID, ClientIP, COUNT(*) AS c, SUM(IsRefresh), AVG(ResolutionWidth) FROM hits WHERE SearchPhrase <> '' GROUP BY SearchEngineID, ClientIP ORDER BY c DESC LIMIT 10;
Time: 20374,998 ms (00:20,375)
Time: 20927,685 ms (00:20,928)
Time: 20843,445 ms (00:20,843)
3
SELECT WatchID, ClientIP, COUNT(*) AS c, SUM(IsRefresh), AVG(ResolutionWidth) FROM hits WHERE SearchPhrase <> '' GROUP BY WatchID, ClientIP ORDER BY c DESC LIMIT 10;
Time: 20431,085 ms (00:20,431)
Time: 21210,669 ms (00:21,211)
Time: 21160,396 ms (00:21,160)
3
SELECT WatchID, ClientIP, COUNT(*) AS c, SUM(IsRefresh), AVG(ResolutionWidth) FROM hits GROUP BY WatchID, ClientIP ORDER BY c DESC LIMIT 10;
Time: 156271,792 ms (02:36,272)
Time: 156166,842 ms (02:36,167)
Time: 153467,977 ms (02:33,468)
3
SELECT URL, COUNT(*) AS c FROM hits GROUP BY URL ORDER BY c DESC LIMIT 10;
Time: 80402,463 ms (01:20,402)
Time: 79791,044 ms (01:19,791)
Time: 81710,559 ms (01:21,711)
3
SELECT 1, URL, COUNT(*) AS c FROM hits GROUP BY 1, URL ORDER BY c DESC LIMIT 10;
Time: 80257,693 ms (01:20,258)
Time: 80214,099 ms (01:20,214)
Time: 81129,213 ms (01:21,129)
3
SELECT ClientIP, ClientIP - 1, ClientIP - 2, ClientIP - 3, COUNT(*) AS c FROM hits GROUP BY ClientIP, ClientIP - 1, ClientIP - 2, ClientIP - 3 ORDER BY c DESC LIMIT 10;
Time: 25868,373 ms (00:25,868)
Time: 25563,769 ms (00:25,564)
Time: 26954,113 ms (00:26,954)
3
SELECT URL, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND DontCountHits = 0 AND IsRefresh = 0 AND URL <> '' GROUP BY URL ORDER BY PageViews DESC LIMIT 10;
Time: 15879,879 ms (00:15,880)
Time: 16383,504 ms (00:16,384)
Time: 16232,127 ms (00:16,232)
3
SELECT Title, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND DontCountHits = 0 AND IsRefresh = 0 AND Title <> '' GROUP BY Title ORDER BY PageViews DESC LIMIT 10;
Time: 15682,023 ms (00:15,682)
Time: 16303,680 ms (00:16,304)
Time: 16419,882 ms (00:16,420)
3
SELECT URL, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND IsRefresh = 0 AND IsLink <> 0 AND IsDownload = 0 GROUP BY URL ORDER BY PageViews DESC LIMIT 10 OFFSET 1000;
Time: 14915,326 ms (00:14,915)
Time: 15669,615 ms (00:15,670)
Time: 15491,437 ms (00:15,491)
3
SELECT TraficSourceID, SearchEngineID, AdvEngineID, CASE WHEN (SearchEngineID = 0 AND AdvEngineID = 0) THEN Referer ELSE '' END AS Src, URL AS Dst, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND IsRefresh = 0 GROUP BY TraficSourceID, SearchEngineID, AdvEngineID, Src, Dst ORDER BY PageViews DESC LIMIT 10 OFFSET 1000;
Time: 14158,395 ms (00:14,158)
Time: 14595,254 ms (00:14,595)
Time: 14599,090 ms (00:14,599)
3
SELECT URLHash, EventDate, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND IsRefresh = 0 AND TraficSourceID IN (-1, 6) AND RefererHash = 3594120000172545465 GROUP BY URLHash, EventDate ORDER BY PageViews DESC LIMIT 10 OFFSET 100;
Time: 18341,903 ms (00:18,342)
Time: 18867,061 ms (00:18,867)
Time: 18533,939 ms (00:18,534)
3
SELECT WindowClientWidth, WindowClientHeight, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND IsRefresh = 0 AND DontCountHits = 0 AND URLHash = 2868770270353813622 GROUP BY WindowClientWidth, WindowClientHeight ORDER BY PageViews DESC LIMIT 10 OFFSET 10000;
Time: 18090,022 ms (00:18,090)
Time: 18690,848 ms (00:18,691)
Time: 18471,913 ms (00:18,472)
3
SELECT DATE_TRUNC('minute', EventTime) AS M, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-14' AND EventDate <= '2013-07-15' AND IsRefresh = 0 AND DontCountHits = 0 GROUP BY DATE_TRUNC('minute', EventTime) ORDER BY DATE_TRUNC('minute', EventTime) LIMIT 10 OFFSET 1000;
Time: 15213,652 ms (00:15,214)
Time: 15760,627 ms (00:15,761)
Time: 15636,045 ms (00:15,636)

  ```
</details>

<details>
  <summary>Release with LTO but Instrumented for PGO</summary>
  
  ```
  3
SELECT COUNT(*) FROM hits;
Time: 13075,090 ms (00:13,075)
3
SELECT COUNT(*) FROM hits WHERE AdvEngineID <> 0;
Time: 15245,449 ms (00:15,245)
3
SELECT SUM(AdvEngineID), COUNT(*), AVG(ResolutionWidth) FROM hits;
Time: 16430,322 ms (00:16,430)
3
SELECT AVG(UserID) FROM hits;
Time: 13760,763 ms (00:13,761)
3
SELECT COUNT(DISTINCT UserID) FROM hits;
Time: 47201,381 ms (00:47,201)
3
SELECT COUNT(DISTINCT SearchPhrase) FROM hits;
Time: 81192,673 ms (01:21,193)
3
SELECT MIN(EventDate), MAX(EventDate) FROM hits;
Time: 13521,136 ms (00:13,521)
3
SELECT AdvEngineID, COUNT(*) FROM hits WHERE AdvEngineID <> 0 GROUP BY AdvEngineID ORDER BY COUNT(*) DESC;
Time: 15429,367 ms (00:15,429)
3
SELECT RegionID, COUNT(DISTINCT UserID) AS u FROM hits GROUP BY RegionID ORDER BY u DESC LIMIT 10;
Time: 47218,950 ms (00:47,219)
3
SELECT RegionID, SUM(AdvEngineID), COUNT(*) AS c, AVG(ResolutionWidth), COUNT(DISTINCT UserID) FROM hits GROUP BY RegionID ORDER BY c DESC LIMIT 10;
Time: 43412,113 ms (00:43,412)
3
SELECT MobilePhoneModel, COUNT(DISTINCT UserID) AS u FROM hits WHERE MobilePhoneModel <> '' GROUP BY MobilePhoneModel ORDER BY u DESC LIMIT 10;
Time: 17692,624 ms (00:17,693)
3
SELECT MobilePhone, MobilePhoneModel, COUNT(DISTINCT UserID) AS u FROM hits WHERE MobilePhoneModel <> '' GROUP BY MobilePhone, MobilePhoneModel ORDER BY u DESC LIMIT 10;
Time: 19076,700 ms (00:19,077)
3
SELECT SearchPhrase, COUNT(*) AS c FROM hits WHERE SearchPhrase <> '' GROUP BY SearchPhrase ORDER BY c DESC LIMIT 10;
Time: 33592,738 ms (00:33,593)
3
SELECT SearchPhrase, COUNT(DISTINCT UserID) AS u FROM hits WHERE SearchPhrase <> '' GROUP BY SearchPhrase ORDER BY u DESC LIMIT 10;
Time: 27404,195 ms (00:27,404)
3
SELECT SearchEngineID, SearchPhrase, COUNT(*) AS c FROM hits WHERE SearchPhrase <> '' GROUP BY SearchEngineID, SearchPhrase ORDER BY c DESC LIMIT 10;
Time: 35065,373 ms (00:35,065)
3
SELECT UserID, COUNT(*) FROM hits GROUP BY UserID ORDER BY COUNT(*) DESC LIMIT 10;
Time: 64806,192 ms (01:04,806)
3
SELECT UserID, SearchPhrase, COUNT(*) FROM hits GROUP BY UserID, SearchPhrase ORDER BY COUNT(*) DESC LIMIT 10;
Time: 60112,229 ms (01:00,112)
3
SELECT UserID, SearchPhrase, COUNT(*) FROM hits GROUP BY UserID, SearchPhrase LIMIT 10;
Time: 48435,282 ms (00:48,435)
3
SELECT UserID, extract(minute FROM EventTime) AS m, SearchPhrase, COUNT(*) FROM hits GROUP BY UserID, m, SearchPhrase ORDER BY COUNT(*) DESC LIMIT 10;
Time: 70843,246 ms (01:10,843)
3
SELECT UserID FROM hits WHERE UserID = 435090932899640449;
Time: 13590,040 ms (00:13,590)
3
SELECT COUNT(*) FROM hits WHERE URL LIKE '%google%';
Time: 16540,619 ms (00:16,541)
3
SELECT SearchPhrase, MIN(URL), COUNT(*) AS c FROM hits WHERE URL LIKE '%google%' AND SearchPhrase <> '' GROUP BY SearchPhrase ORDER BY c DESC LIMIT 10;
Time: 18513,578 ms (00:18,514)
3
SELECT SearchPhrase, MIN(URL), MIN(Title), COUNT(*) AS c, COUNT(DISTINCT UserID) FROM hits WHERE Title LIKE '%Google%' AND URL NOT LIKE '%.google.%' AND SearchPhrase <> '' GROUP BY SearchPhrase ORDER BY c DESC LIMIT 10;
Time: 18544,911 ms (00:18,545)
3
SELECT * FROM hits WHERE URL LIKE '%google%' ORDER BY EventTime LIMIT 10;
Time: 16454,000 ms (00:16,454)
3
SELECT SearchPhrase FROM hits WHERE SearchPhrase <> '' ORDER BY EventTime LIMIT 10;
Time: 16071,716 ms (00:16,072)
3
SELECT SearchPhrase FROM hits WHERE SearchPhrase <> '' ORDER BY SearchPhrase LIMIT 10;
Time: 16205,746 ms (00:16,206)
3
SELECT SearchPhrase FROM hits WHERE SearchPhrase <> '' ORDER BY EventTime, SearchPhrase LIMIT 10;
Time: 16115,563 ms (00:16,116)
3
SELECT CounterID, AVG(length(URL)) AS l, COUNT(*) AS c FROM hits WHERE URL <> '' GROUP BY CounterID HAVING COUNT(*) > 100000 ORDER BY l DESC LIMIT 25;
Time: 21066,883 ms (00:21,067)
3
SELECT REGEXP_REPLACE(Referer, '^https?://(?:www.)?([^/]+)/.*$', '1') AS k, AVG(length(Referer)) AS l, COUNT(*) AS c, MIN(Referer) FROM hits WHERE Referer <> '' GROUP BY k HAVING COUNT(*) > 100000 ORDER BY l DESC LIMIT 25;
Time: 85472,122 ms (01:25,472)
3
SELECT SUM(ResolutionWidth), SUM(ResolutionWidth + 1), SUM(ResolutionWidth + 2), SUM(ResolutionWidth + 3), SUM(ResolutionWidth + 4), SUM(ResolutionWidth + 5), SUM(ResolutionWidth + 6), SUM(ResolutionWidth + 7), SUM(ResolutionWidth + 8), SUM(ResolutionWidth + 9), SUM(ResolutionWidth + 10), SUM(ResolutionWidth + 11), SUM(ResolutionWidth + 12), SUM(ResolutionWidth + 13), SUM(ResolutionWidth + 14), SUM(ResolutionWidth + 15), SUM(ResolutionWidth + 16), SUM(ResolutionWidth + 17), SUM(ResolutionWidth + 18), SUM(ResolutionWidth + 19), SUM(ResolutionWidth + 20), SUM(ResolutionWidth + 21), SUM(ResolutionWidth + 22), SUM(ResolutionWidth + 23), SUM(ResolutionWidth + 24), SUM(ResolutionWidth + 25), SUM(ResolutionWidth + 26), SUM(ResolutionWidth + 27), SUM(ResolutionWidth + 28), SUM(ResolutionWidth + 29), SUM(ResolutionWidth + 30), SUM(ResolutionWidth + 31), SUM(ResolutionWidth + 32), SUM(ResolutionWidth + 33), SUM(ResolutionWidth + 34), SUM(ResolutionWidth + 35), SUM(ResolutionWidth + 36), SUM(ResolutionWidth + 37), SUM(ResolutionWidth + 38), SUM(ResolutionWidth + 39), SUM(ResolutionWidth + 40), SUM(ResolutionWidth + 41), SUM(ResolutionWidth + 42), SUM(ResolutionWidth + 43), SUM(ResolutionWidth + 44), SUM(ResolutionWidth + 45), SUM(ResolutionWidth + 46), SUM(ResolutionWidth + 47), SUM(ResolutionWidth + 48), SUM(ResolutionWidth + 49), SUM(ResolutionWidth + 50), SUM(ResolutionWidth + 51), SUM(ResolutionWidth + 52), SUM(ResolutionWidth + 53), SUM(ResolutionWidth + 54), SUM(ResolutionWidth + 55), SUM(ResolutionWidth + 56), SUM(ResolutionWidth + 57), SUM(ResolutionWidth + 58), SUM(ResolutionWidth + 59), SUM(ResolutionWidth + 60), SUM(ResolutionWidth + 61), SUM(ResolutionWidth + 62), SUM(ResolutionWidth + 63), SUM(ResolutionWidth + 64), SUM(ResolutionWidth + 65), SUM(ResolutionWidth + 66), SUM(ResolutionWidth + 67), SUM(ResolutionWidth + 68), SUM(ResolutionWidth + 69), SUM(ResolutionWidth + 70), SUM(ResolutionWidth + 71), SUM(ResolutionWidth + 72), SUM(ResolutionWidth + 73), SUM(ResolutionWidth + 74), SUM(ResolutionWidth + 75), SUM(ResolutionWidth + 76), SUM(ResolutionWidth + 77), SUM(ResolutionWidth + 78), SUM(ResolutionWidth + 79), SUM(ResolutionWidth + 80), SUM(ResolutionWidth + 81), SUM(ResolutionWidth + 82), SUM(ResolutionWidth + 83), SUM(ResolutionWidth + 84), SUM(ResolutionWidth + 85), SUM(ResolutionWidth + 86), SUM(ResolutionWidth + 87), SUM(ResolutionWidth + 88), SUM(ResolutionWidth + 89) FROM hits;
Time: 44180,615 ms (00:44,181)
3
SELECT SearchEngineID, ClientIP, COUNT(*) AS c, SUM(IsRefresh), AVG(ResolutionWidth) FROM hits WHERE SearchPhrase <> '' GROUP BY SearchEngineID, ClientIP ORDER BY c DESC LIMIT 10;
Time: 24932,729 ms (00:24,933)
3
SELECT WatchID, ClientIP, COUNT(*) AS c, SUM(IsRefresh), AVG(ResolutionWidth) FROM hits WHERE SearchPhrase <> '' GROUP BY WatchID, ClientIP ORDER BY c DESC LIMIT 10;
Time: 25833,619 ms (00:25,834)
3
SELECT WatchID, ClientIP, COUNT(*) AS c, SUM(IsRefresh), AVG(ResolutionWidth) FROM hits GROUP BY WatchID, ClientIP ORDER BY c DESC LIMIT 10;
Time: 226593,279 ms (03:46,593)
3
SELECT URL, COUNT(*) AS c FROM hits GROUP BY URL ORDER BY c DESC LIMIT 10;
Time: 110944,857 ms (01:50,945)
3
SELECT 1, URL, COUNT(*) AS c FROM hits GROUP BY 1, URL ORDER BY c DESC LIMIT 10;
Time: 111000,568 ms (01:51,001)
3
SELECT ClientIP, ClientIP - 1, ClientIP - 2, ClientIP - 3, COUNT(*) AS c FROM hits GROUP BY ClientIP, ClientIP - 1, ClientIP - 2, ClientIP - 3 ORDER BY c DESC LIMIT 10;
Time: 31446,405 ms (00:31,446)
3
SELECT URL, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND DontCountHits = 0 AND IsRefresh = 0 AND URL <> '' GROUP BY URL ORDER BY PageViews DESC LIMIT 10;
Time: 18376,261 ms (00:18,376)
3
SELECT Title, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND DontCountHits = 0 AND IsRefresh = 0 AND Title <> '' GROUP BY Title ORDER BY PageViews DESC LIMIT 10;
Time: 18108,667 ms (00:18,109)
3
SELECT URL, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND IsRefresh = 0 AND IsLink <> 0 AND IsDownload = 0 GROUP BY URL ORDER BY PageViews DESC LIMIT 10 OFFSET 1000;
Time: 16895,396 ms (00:16,895)
3
SELECT TraficSourceID, SearchEngineID, AdvEngineID, CASE WHEN (SearchEngineID = 0 AND AdvEngineID = 0) THEN Referer ELSE '' END AS Src, URL AS Dst, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND IsRefresh = 0 GROUP BY TraficSourceID, SearchEngineID, AdvEngineID, Src, Dst ORDER BY PageViews DESC LIMIT 10 OFFSET 1000;
Time: 15455,092 ms (00:15,455)
3
SELECT URLHash, EventDate, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND IsRefresh = 0 AND TraficSourceID IN (-1, 6) AND RefererHash = 3594120000172545465 GROUP BY URLHash, EventDate ORDER BY PageViews DESC LIMIT 10 OFFSET 100;
Time: 22039,957 ms (00:22,040)
3
SELECT WindowClientWidth, WindowClientHeight, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND IsRefresh = 0 AND DontCountHits = 0 AND URLHash = 2868770270353813622 GROUP BY WindowClientWidth, WindowClientHeight ORDER BY PageViews DESC LIMIT 10 OFFSET 10000;
Time: 21352,692 ms (00:21,353)
3
SELECT DATE_TRUNC('minute', EventTime) AS M, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-14' AND EventDate <= '2013-07-15' AND IsRefresh = 0 AND DontCountHits = 0 GROUP BY DATE_TRUNC('minute', EventTime) ORDER BY DATE_TRUNC('minute', EventTime) LIMIT 10 OFFSET 1000;
Time: 17209,025 ms (00:17,209)


  ```
</details>

<details>
  <summary>Release with LTO and AutoFDO</summary>
  
  ```
  3
SELECT COUNT(*) FROM hits;
Time: 13007,926 ms (00:13,008)
Time: 13227,983 ms (00:13,228)
Time: 13203,369 ms (00:13,203)
3
SELECT COUNT(*) FROM hits WHERE AdvEngineID <> 0;
Time: 15115,845 ms (00:15,116)
Time: 15612,447 ms (00:15,612)
Time: 15412,048 ms (00:15,412)
3
SELECT SUM(AdvEngineID), COUNT(*), AVG(ResolutionWidth) FROM hits;
Time: 15094,784 ms (00:15,095)
Time: 16066,922 ms (00:16,067)
Time: 15822,866 ms (00:15,823)
3
SELECT AVG(UserID) FROM hits;
Time: 13337,034 ms (00:13,337)
Time: 13706,869 ms (00:13,707)
Time: 13705,519 ms (00:13,706)
3
SELECT COUNT(DISTINCT UserID) FROM hits;
Time: 42645,104 ms (00:42,645)
Time: 42552,459 ms (00:42,552)
Time: 43466,180 ms (00:43,466)
3
SELECT COUNT(DISTINCT SearchPhrase) FROM hits;
Time: 73096,652 ms (01:13,097)
Time: 74500,208 ms (01:14,500)
Time: 75175,203 ms (01:15,175)
3
SELECT MIN(EventDate), MAX(EventDate) FROM hits;
Time: 13493,350 ms (00:13,493)
Time: 13714,967 ms (00:13,715)
Time: 13394,134 ms (00:13,394)
3
SELECT AdvEngineID, COUNT(*) FROM hits WHERE AdvEngineID <> 0 GROUP BY AdvEngineID ORDER BY COUNT(*) DESC;
Time: 14964,617 ms (00:14,965)
Time: 15591,637 ms (00:15,592)
Time: 15519,279 ms (00:15,519)
3
SELECT RegionID, COUNT(DISTINCT UserID) AS u FROM hits GROUP BY RegionID ORDER BY u DESC LIMIT 10;
Time: 32883,874 ms (00:32,884)
Time: 33779,543 ms (00:33,780)
Time: 33703,483 ms (00:33,703)
3
SELECT RegionID, SUM(AdvEngineID), COUNT(*) AS c, AVG(ResolutionWidth), COUNT(DISTINCT UserID) FROM hits GROUP BY RegionID ORDER BY c DESC LIMIT 10;
Time: 38623,406 ms (00:38,623)
Time: 38962,959 ms (00:38,963)
Time: 39030,108 ms (00:39,030)
3
SELECT MobilePhoneModel, COUNT(DISTINCT UserID) AS u FROM hits WHERE MobilePhoneModel <> '' GROUP BY MobilePhoneModel ORDER BY u DESC LIMIT 10;
Time: 16667,210 ms (00:16,667)
Time: 17204,784 ms (00:17,205)
Time: 17050,033 ms (00:17,050)
3
SELECT MobilePhone, MobilePhoneModel, COUNT(DISTINCT UserID) AS u FROM hits WHERE MobilePhoneModel <> '' GROUP BY MobilePhone, MobilePhoneModel ORDER BY u DESC LIMIT 10;
Time: 17112,780 ms (00:17,113)
Time: 17693,871 ms (00:17,694)
Time: 17714,126 ms (00:17,714)
3
SELECT SearchPhrase, COUNT(*) AS c FROM hits WHERE SearchPhrase <> '' GROUP BY SearchPhrase ORDER BY c DESC LIMIT 10;
Time: 26899,869 ms (00:26,900)
Time: 27495,367 ms (00:27,495)
Time: 27768,677 ms (00:27,769)
3
SELECT SearchPhrase, COUNT(DISTINCT UserID) AS u FROM hits WHERE SearchPhrase <> '' GROUP BY SearchPhrase ORDER BY u DESC LIMIT 10;
Time: 24828,139 ms (00:24,828)
Time: 25567,001 ms (00:25,567)
Time: 25476,110 ms (00:25,476)
3
SELECT SearchEngineID, SearchPhrase, COUNT(*) AS c FROM hits WHERE SearchPhrase <> '' GROUP BY SearchEngineID, SearchPhrase ORDER BY c DESC LIMIT 10;
Time: 30124,947 ms (00:30,125)
Time: 30956,472 ms (00:30,956)
Time: 31051,207 ms (00:31,051)
3
SELECT UserID, COUNT(*) FROM hits GROUP BY UserID ORDER BY COUNT(*) DESC LIMIT 10;
Time: 50350,756 ms (00:50,351)
Time: 50391,713 ms (00:50,392)
Time: 50115,083 ms (00:50,115)
3
SELECT UserID, SearchPhrase, COUNT(*) FROM hits GROUP BY UserID, SearchPhrase ORDER BY COUNT(*) DESC LIMIT 10;
Time: 50150,619 ms (00:50,151)
Time: 50641,829 ms (00:50,642)
Time: 50216,764 ms (00:50,217)
3
SELECT UserID, SearchPhrase, COUNT(*) FROM hits GROUP BY UserID, SearchPhrase LIMIT 10;
Time: 39430,524 ms (00:39,431)
Time: 40033,486 ms (00:40,033)
Time: 39613,634 ms (00:39,614)
3
SELECT UserID, extract(minute FROM EventTime) AS m, SearchPhrase, COUNT(*) FROM hits GROUP BY UserID, m, SearchPhrase ORDER BY COUNT(*) DESC LIMIT 10;
Time: 58999,727 ms (00:59,000)
Time: 58791,863 ms (00:58,792)
Time: 59352,497 ms (00:59,352)
3
SELECT UserID FROM hits WHERE UserID = 435090932899640449;
Time: 13253,942 ms (00:13,254)
Time: 13444,281 ms (00:13,444)
Time: 13485,875 ms (00:13,486)
3
SELECT COUNT(*) FROM hits WHERE URL LIKE '%google%';
Time: 15431,326 ms (00:15,431)
Time: 15992,689 ms (00:15,993)
Time: 15939,704 ms (00:15,940)
3
SELECT SearchPhrase, MIN(URL), COUNT(*) AS c FROM hits WHERE URL LIKE '%google%' AND SearchPhrase <> '' GROUP BY SearchPhrase ORDER BY c DESC LIMIT 10;
Time: 17534,220 ms (00:17,534)
Time: 18069,795 ms (00:18,070)
Time: 17960,348 ms (00:17,960)
3
SELECT SearchPhrase, MIN(URL), MIN(Title), COUNT(*) AS c, COUNT(DISTINCT UserID) FROM hits WHERE Title LIKE '%Google%' AND URL NOT LIKE '%.google.%' AND SearchPhrase <> '' GROUP BY SearchPhrase ORDER BY c DESC LIMIT 10;
Time: 18011,061 ms (00:18,011)
Time: 18527,253 ms (00:18,527)
Time: 18324,606 ms (00:18,325)
3
SELECT * FROM hits WHERE URL LIKE '%google%' ORDER BY EventTime LIMIT 10;
Time: 15557,686 ms (00:15,558)
Time: 15953,945 ms (00:15,954)
Time: 15903,131 ms (00:15,903)
3
SELECT SearchPhrase FROM hits WHERE SearchPhrase <> '' ORDER BY EventTime LIMIT 10;
Time: 15446,827 ms (00:15,447)
Time: 15936,618 ms (00:15,937)
Time: 15979,374 ms (00:15,979)
3
SELECT SearchPhrase FROM hits WHERE SearchPhrase <> '' ORDER BY SearchPhrase LIMIT 10;
Time: 15394,003 ms (00:15,394)
Time: 16110,094 ms (00:16,110)
Time: 15932,535 ms (00:15,933)
3
SELECT SearchPhrase FROM hits WHERE SearchPhrase <> '' ORDER BY EventTime, SearchPhrase LIMIT 10;
Time: 15354,537 ms (00:15,355)
Time: 15792,004 ms (00:15,792)
Time: 15867,547 ms (00:15,868)
3
SELECT CounterID, AVG(length(URL)) AS l, COUNT(*) AS c FROM hits WHERE URL <> '' GROUP BY CounterID HAVING COUNT(*) > 100000 ORDER BY l DESC LIMIT 25;
Time: 18685,794 ms (00:18,686)
Time: 19203,143 ms (00:19,203)
Time: 19140,114 ms (00:19,140)
3
SELECT REGEXP_REPLACE(Referer, '^https?://(?:www.)?([^/]+)/.*$', '1') AS k, AVG(length(Referer)) AS l, COUNT(*) AS c, MIN(Referer) FROM hits WHERE Referer <> '' GROUP BY k HAVING COUNT(*) > 100000 ORDER BY l DESC LIMIT 25;
Time: 72722,038 ms (01:12,722)
Time: 72809,808 ms (01:12,810)
Time: 72404,802 ms (01:12,405)
3
SELECT SUM(ResolutionWidth), SUM(ResolutionWidth + 1), SUM(ResolutionWidth + 2), SUM(ResolutionWidth + 3), SUM(ResolutionWidth + 4), SUM(ResolutionWidth + 5), SUM(ResolutionWidth + 6), SUM(ResolutionWidth + 7), SUM(ResolutionWidth + 8), SUM(ResolutionWidth + 9), SUM(ResolutionWidth + 10), SUM(ResolutionWidth + 11), SUM(ResolutionWidth + 12), SUM(ResolutionWidth + 13), SUM(ResolutionWidth + 14), SUM(ResolutionWidth + 15), SUM(ResolutionWidth + 16), SUM(ResolutionWidth + 17), SUM(ResolutionWidth + 18), SUM(ResolutionWidth + 19), SUM(ResolutionWidth + 20), SUM(ResolutionWidth + 21), SUM(ResolutionWidth + 22), SUM(ResolutionWidth + 23), SUM(ResolutionWidth + 24), SUM(ResolutionWidth + 25), SUM(ResolutionWidth + 26), SUM(ResolutionWidth + 27), SUM(ResolutionWidth + 28), SUM(ResolutionWidth + 29), SUM(ResolutionWidth + 30), SUM(ResolutionWidth + 31), SUM(ResolutionWidth + 32), SUM(ResolutionWidth + 33), SUM(ResolutionWidth + 34), SUM(ResolutionWidth + 35), SUM(ResolutionWidth + 36), SUM(ResolutionWidth + 37), SUM(ResolutionWidth + 38), SUM(ResolutionWidth + 39), SUM(ResolutionWidth + 40), SUM(ResolutionWidth + 41), SUM(ResolutionWidth + 42), SUM(ResolutionWidth + 43), SUM(ResolutionWidth + 44), SUM(ResolutionWidth + 45), SUM(ResolutionWidth + 46), SUM(ResolutionWidth + 47), SUM(ResolutionWidth + 48), SUM(ResolutionWidth + 49), SUM(ResolutionWidth + 50), SUM(ResolutionWidth + 51), SUM(ResolutionWidth + 52), SUM(ResolutionWidth + 53), SUM(ResolutionWidth + 54), SUM(ResolutionWidth + 55), SUM(ResolutionWidth + 56), SUM(ResolutionWidth + 57), SUM(ResolutionWidth + 58), SUM(ResolutionWidth + 59), SUM(ResolutionWidth + 60), SUM(ResolutionWidth + 61), SUM(ResolutionWidth + 62), SUM(ResolutionWidth + 63), SUM(ResolutionWidth + 64), SUM(ResolutionWidth + 65), SUM(ResolutionWidth + 66), SUM(ResolutionWidth + 67), SUM(ResolutionWidth + 68), SUM(ResolutionWidth + 69), SUM(ResolutionWidth + 70), SUM(ResolutionWidth + 71), SUM(ResolutionWidth + 72), SUM(ResolutionWidth + 73), SUM(ResolutionWidth + 74), SUM(ResolutionWidth + 75), SUM(ResolutionWidth + 76), SUM(ResolutionWidth + 77), SUM(ResolutionWidth + 78), SUM(ResolutionWidth + 79), SUM(ResolutionWidth + 80), SUM(ResolutionWidth + 81), SUM(ResolutionWidth + 82), SUM(ResolutionWidth + 83), SUM(ResolutionWidth + 84), SUM(ResolutionWidth + 85), SUM(ResolutionWidth + 86), SUM(ResolutionWidth + 87), SUM(ResolutionWidth + 88), SUM(ResolutionWidth + 89) FROM hits;
Time: 36168,836 ms (00:36,169)
Time: 36527,383 ms (00:36,527)
Time: 36851,984 ms (00:36,852)
3
SELECT SearchEngineID, ClientIP, COUNT(*) AS c, SUM(IsRefresh), AVG(ResolutionWidth) FROM hits WHERE SearchPhrase <> '' GROUP BY SearchEngineID, ClientIP ORDER BY c DESC LIMIT 10;
Time: 22493,051 ms (00:22,493)
Time: 22748,170 ms (00:22,748)
Time: 22842,853 ms (00:22,843)
3
SELECT WatchID, ClientIP, COUNT(*) AS c, SUM(IsRefresh), AVG(ResolutionWidth) FROM hits WHERE SearchPhrase <> '' GROUP BY WatchID, ClientIP ORDER BY c DESC LIMIT 10;
Time: 22555,507 ms (00:22,556)
Time: 23335,974 ms (00:23,336)
Time: 23042,491 ms (00:23,042)
3
SELECT WatchID, ClientIP, COUNT(*) AS c, SUM(IsRefresh), AVG(ResolutionWidth) FROM hits GROUP BY WatchID, ClientIP ORDER BY c DESC LIMIT 10;
Time: 180060,166 ms (03:00,060)
Time: 175794,703 ms (02:55,795)
Time: 180155,889 ms (03:00,156)
3
SELECT URL, COUNT(*) AS c FROM hits GROUP BY URL ORDER BY c DESC LIMIT 10;
Time: 88552,637 ms (01:28,553)
Time: 89677,398 ms (01:29,677)
Time: 88805,311 ms (01:28,805)
3
SELECT 1, URL, COUNT(*) AS c FROM hits GROUP BY 1, URL ORDER BY c DESC LIMIT 10;
Time: 88642,438 ms (01:28,642)
Time: 89734,860 ms (01:29,735)
Time: 89839,944 ms (01:29,840)
3
SELECT ClientIP, ClientIP - 1, ClientIP - 2, ClientIP - 3, COUNT(*) AS c FROM hits GROUP BY ClientIP, ClientIP - 1, ClientIP - 2, ClientIP - 3 ORDER BY c DESC LIMIT 10;
Time: 27910,344 ms (00:27,910)
Time: 28451,516 ms (00:28,452)
Time: 28308,822 ms (00:28,309)
3
SELECT URL, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND DontCountHits = 0 AND IsRefresh = 0 AND URL <> '' GROUP BY URL ORDER BY PageViews DESC LIMIT 10;
Time: 17387,428 ms (00:17,387)
Time: 17907,335 ms (00:17,907)
Time: 17995,836 ms (00:17,996)
3
SELECT Title, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND DontCountHits = 0 AND IsRefresh = 0 AND Title <> '' GROUP BY Title ORDER BY PageViews DESC LIMIT 10;
Time: 17317,700 ms (00:17,318)
Time: 18021,934 ms (00:18,022)
Time: 17543,523 ms (00:17,544)
3
SELECT URL, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND IsRefresh = 0 AND IsLink <> 0 AND IsDownload = 0 GROUP BY URL ORDER BY PageViews DESC LIMIT 10 OFFSET 1000;
Time: 16256,922 ms (00:16,257)
Time: 16774,566 ms (00:16,775)
Time: 16701,961 ms (00:16,702)
3
SELECT TraficSourceID, SearchEngineID, AdvEngineID, CASE WHEN (SearchEngineID = 0 AND AdvEngineID = 0) THEN Referer ELSE '' END AS Src, URL AS Dst, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND IsRefresh = 0 GROUP BY TraficSourceID, SearchEngineID, AdvEngineID, Src, Dst ORDER BY PageViews DESC LIMIT 10 OFFSET 1000;
Time: 14687,581 ms (00:14,688)
Time: 15042,958 ms (00:15,043)
Time: 15125,441 ms (00:15,125)
3
SELECT URLHash, EventDate, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND IsRefresh = 0 AND TraficSourceID IN (-1, 6) AND RefererHash = 3594120000172545465 GROUP BY URLHash, EventDate ORDER BY PageViews DESC LIMIT 10 OFFSET 100;
Time: 20685,592 ms (00:20,686)
Time: 21105,250 ms (00:21,105)
Time: 21216,985 ms (00:21,217)
3
SELECT WindowClientWidth, WindowClientHeight, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-01' AND EventDate <= '2013-07-31' AND IsRefresh = 0 AND DontCountHits = 0 AND URLHash = 2868770270353813622 GROUP BY WindowClientWidth, WindowClientHeight ORDER BY PageViews DESC LIMIT 10 OFFSET 10000;
Time: 20162,071 ms (00:20,162)
Time: 21060,588 ms (00:21,061)
Time: 20768,418 ms (00:20,768)
3
SELECT DATE_TRUNC('minute', EventTime) AS M, COUNT(*) AS PageViews FROM hits WHERE CounterID = 62 AND EventDate >= '2013-07-14' AND EventDate <= '2013-07-15' AND IsRefresh = 0 AND DontCountHits = 0 GROUP BY DATE_TRUNC('minute', EventTime) ORDER BY DATE_TRUNC('minute', EventTime) LIMIT 10 OFFSET 1000;
Time: 16589,529 ms (00:16,590)
Time: 17451,181 ms (00:17,451)
Time: 16999,147 ms (00:16,999)


  ```
</details>

## Conclusions

Definetely, PostgreSQL benefits from LTO and PGO. So if you have an ability to recompile PostgreSQL and you have a need to optimize your PostgreSQL installation - you know what to do.