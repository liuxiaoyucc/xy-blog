---
title: 解决PHP CI(CodeIgniter)框架使用insert_batch()批量插入时unique列冲突导致操作失败的问题
date: 2019-12-27 23:32:39
categories:
  - - 大熊BIGBEAR
    - PHP
    - CI
tags:
  - PHP
  - CI
  - 大熊BIGBEAR
---

<!-- <meta name="referrer" content="no-referrer" /> -->

>在CI中, 有时我们想将很多条数据, 批量插入到数据库中, 我们知道有[insert_batch()](https://codeigniter.org.cn/user_guide/database/query_builder.html)这个函数, 但是如果insert_batch时, 遇到了mysql中的UNIQUE约束, 就会导致插入失败, 并回滚, 那么我们该如何让其跳过冲突的列, 并继续执行下面的数据呢?
 <!-- more -->
 
既然insert_batch()满足不了我们的需求, 那我们就自己造一个, 首先我们找到insert_batch()函数的位置, 看下源码,
```
# CI_VERSION = 3.1.11, 这个版本可以在system/core/CodeIgniter.php中查看
# 位置在 system/database/DB_query_builder.php insert_batch()
/**
 * Insert_Batch
 *
 * Compiles batch insert strings and runs the queries
 *
 * @param	string	$table	Table to insert into
 * @param	array	$set 	An associative array of insert values
 * @param	bool	$escape	Whether to escape values and identifiers
 * @return	int	Number of rows inserted or FALSE on failure
 */
public function insert_batch($table, $set = NULL, $escape = NULL, $batch_size = 100)
{
	if ($set === NULL)
	{
		if (empty($this->qb_set))
		{
			return ($this->db_debug) ? $this->display_error('db_must_use_set') : FALSE;
		}
	}
	else
	{
		if (empty($set))
		{
			return ($this->db_debug) ? $this->display_error('insert_batch() called with no data') : FALSE;
		}

		$this->set_insert_batch($set, '', $escape);
	}

	if (strlen($table) === 0)
	{
		if ( ! isset($this->qb_from[0]))
		{
			return ($this->db_debug) ? $this->display_error('db_must_set_table') : FALSE;
		}

		$table = $this->qb_from[0];
	}

	// Batch this baby
	$affected_rows = 0;
	for ($i = 0, $total = count($this->qb_set); $i < $total; $i += $batch_size)
	{
		if ($this->query($this->_insert_batch($this->protect_identifiers($table, TRUE, $escape, FALSE), $this->qb_keys, array_slice($this->qb_set, $i, $batch_size))))
		{
			$affected_rows += $this->affected_rows();
		}
	}

	$this->_reset_write();
	return $affected_rows;
}

```
可以看到, _insert_batch函数生成了sql语句, 最终由query函数执行, 并返回影响条数, 现在我们使用__IGNORE__关键字, 来改写一下sql语句那里就ok了, 为了不影响源码, 我们在insert_batch()下面新写一个insert_ignore_batch函数, 前面都省略, 直接从for循环这里开始
```
/*
* insert_batch() using INSERT IGNORE INTO instead of INSERT INTO
* @ xyliu
*/
public function insert_ignore_batch($table, $set = NULL, $escape = NULL, $batch_size = 100)
{
    // ...这里的内容直接复制过来, 保持原样就可以, 这里太占篇幅被我省略掉了

    // Batch this baby;
    $affected_rows = 0;
    for ($i = 0, $total = count($this->qb_set); $i < $total; $i = $i + 100)
    {
        $sql = $this->_insert_batch($this->protect_identifiers($table, TRUE, $escape, FALSE), $this->qb_keys, array_slice($this->qb_set, $i, $batch_size));
        $sql = str_replace('INSERT INTO','INSERT IGNORE INTO',$sql); //忽略错误
        if ($this->query($sql)) {
        	$affected_rows += $this->affected_rows();
        }
    }

    $this->_reset_write();
    return $affected_rows;;
}

```

我们在使用的时候可以直接像使用insert_batch()那样使用insert_ignore_batch
```
$this->db->insert_ignore_batch();
```