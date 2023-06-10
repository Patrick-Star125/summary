## LAB1

In lab1, you should implement two estimation methods, a query-driven method based on regression models like neural networks and a data-driven method based on SPNs.

### The Input

In this lab, we use `IMDB.title` to train and test our models. All data is in `data` directory and it consists of:

- `title_schema.sql`: the schema of the table `title`;
- `query_train_*.json`: queries used to train your model;
- `query_test_*.json`: queries used to test your model;
- `title_sample_*.csv`: sample data used to construct your SPN;
- `title_stats.json`: statistics of `title` exported from TiDB;

For simplicity, all queries we used in lab1 are well formatted range queries with the same pattern `select * from imdb.title where c1>? and c1<? and c2>? and c2<? and ...`.

And all columns that appear in our queries are INT column, more specifically, you can only consider `kind_id`, `production_year`, `imdb_id`, `episode_of_id`, `season_nr`, `episode_nr` columns in lab1.

These files are supposed to be used as below:

![](http://1.14.100.228:8002/images/2022/11/24/20221124103227.png)

### The Code

In `range_query.py`, we've implemented some functions that can help you parse range queries well.

In `statistics.py`, we've implemented some structure related to statistics. `TableStats` is the abstraction of a table's statistics, which consists of multiple columns. A column's statistics is stored in a `Histogram` and a `TopN`, and they can do estimation for single-column predicates well. For multi-column predicates estimation, some estimators based on different strategies(AVI, ExpBackOff and MinSel) can be used.

You need to fill some missing code in `learn_from_query.py`, `learn_from_data.py` and `statistics.py`. Missing code is marked with comment `YOUR CODE HERE`.

After you fill all missing code, you can run `python evaluation.py` and it will test your models and generate a report under `eval` directory.

And a file `eval/results.json` will be generated, please push it to github to trigger auto-grading.