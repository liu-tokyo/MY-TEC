# for update 和 for update nowait的区别

for update 和for update nowait 的区别：

1. 首先一点，如果只是select的话，Oracle是不会加任何锁的，也就是Oracle对select读到的数据不会有任何限制，虽然这时候有可能另外一个进程正在修改表中的数据，并且修改的结果可能影响到你目前select语句的结果，但是因为没有锁，所以select结果为当前时刻表中记录的状态。

2. 如果加入了for update，则Oracle一旦发现(符合查询条件的)这批数据正在被修改，则不会发出该select语句查询，直到数据被修改结束(被commit)，马上自动执行这个select语句。同样，如果该查询语句发出后，有人需要修改这批数据(中的一条或几条)，它也必须等到查询结束后(commit)后，才能修改。

3. for update nowait和for update 都会对所查询到得结果集进行加锁，所不同的是，如果另外一个线程正在修改结果集中的数据，for update nowait 不会进行资源等待，只要发现结果集中有些数据被加锁，立刻返回“ORA-00054错误，内容是资源正忙， 但指定以NOWAIT 方式获取资源”。

4. for update 和for update nowait 加上的是一个**行级锁**，也就是只有符合 **where** 条件的数据被加锁。如果仅仅用 **update** 语句来更改数据时，可能会因为加不上锁而没有响应地、莫名其妙地等待，但如果在此之前，for update NOWAIT语句将要更改的数据试探性地加锁，就可以通过立即返回的错误提示而明白其中的道理，或许这就是for Update和NOWAIT的意义之所在。

5. 经过测试，以for update 或for update nowait方式进行查询加锁，在select的结果集中，只要有任何一个记录在加锁，则整个结果集都在等待系统资源(如果是nowait，则抛出相应的异常)。

6. for update的作用是用于对选择的行加**排他锁**的，在有些情况下，事务的处理需要先选中一些记录，再对这些记录进行处理。因此需要排他锁。
   而for update nowait的作用与for update相同，不同的是其他事务申请被锁定的行数据时是等待该事务释放资源，还是直接返回无法获得资源。