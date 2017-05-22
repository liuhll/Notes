更新大批量数据的方法
=========================

原始语句::

   -- 原始语句 ----------------------------------------------------------------------------------------
   --UPDATE a SET a.Col_442 = b.ManageCode --收件地编码
   --FROM TA_Logistics a 
   --JOIN TB_CompanyJJ b ON b.Company = a.Col_051
   --WHERE Col_076 >= '2016-06-01' AND Col_076 < '2016-11-02' AND (Col_063 = '' OR Col_063 IS NOT NULL)
   --	AND a.Col_442 IS NULL


等效方法

- 这个方法不会造成大面积阻塞，也就不会造成大面积卡顿。理解：**本质上是将一个大事务分解为多个小事务。所以以下语句不能有外层事务**

::

   SELECT a.sys_guid/*待修改表的主键*/, ManageCode/*每行应取的目标值*/
     INTO #tmp
     FROM TA_Logistics a 
   JOIN TB_CompanyJJ b ON b.Company = a.Col_051
   WHERE Col_076 >= '2016-06-01' AND Col_076 < '2016-11-02' AND (Col_063 = '' OR Col_063 IS NOT NULL)
   	AND a.Col_442 IS NULL;
   
   SELECT TOP(0) * INTO #tmp1 FROM #tmp t;
   CREATE CLUSTERED INDEX CX_tmp1 ON #tmp1(sys_guid);
   
   WHILE(1=1)
   BEGIN
   	-- TOP(200) 是用来限定单次循环中修改的数据量
   	DELETE TOP(200) t OUTPUT Deleted.* INTO #tmp1 FROM #tmp t;
   	IF(@@ROWCOUNT = 0) BREAK;
   
   	-- 这里要注意保证更新的行数不会因为关联产生更多。这也是临时表中存的是目标表的主键的原因。
   	UPDATE a SET a.Col_442 = t.ManageCode
   	FROM TA_Logistics a 
   	JOIN #tmp1 t ON a.sys_guid = t.sys_guid;
   
   	TRUNCATE TABLE #tmp1;
   
   	-- 每次循环后等待一小段时间，让其它会话也能干活。这样更不容易引起长时间大面积的阻塞。
   	--	 同时这也有助于降低磁盘繁忙度。
   	WAITFOR DELAY '0:0:0.2';
   END
   
   TRUNCATE TABLE #tmp;
   DROP TABLE #tmp;
   DROP TABLE #tmp1;   