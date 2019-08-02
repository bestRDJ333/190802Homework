# 190802Homework

```mysql
delimiter $$
create procedure pro_orderConfirm(mID int, stID int, bDate datetime)
    declare done int default false;

    declare tmpProductName varchar(20) default "";
    declare qty int default 0;
    declare unitPrice int default 0;
    declare total int default 0;

    -- 取得總共有哪些產品被訂購
    declare curs cursor for 
        select productName from buyWhat 
        where memberID = mID 
        and storeID = stID
        and buildDate = bDate
        group by productName;

	declare continue handler for not found set done = true;

    open curs;
    fetch curs into tmpProductName;

    while not done do
        -- 計算單一產品數量
        set qty = 
            select count(productName) from buyWhat 
            where memberID = mID 
            and storeID = stID
            and buildDate = bDate
            and productName = tmpProductName;
        
        -- 取得單價
        set unitPrice = 
            select productName
            where storeID = stID
            and productName = tmpProductName;

        -- 插入資料
        insert into orderdetail(buildDate, memberID, storeID, productName, quantity)
        values (bDate, mID, stID, tmpProductName, qty);

        -- 更新訂單總價
        set total = total + unitPrice * qty;

        fetch curs into tmpProductName;
    end while;

    close curs;
    -- 訂單內容更新
    update order set totalPrice = total 
    where memberID = mID 
    and storeID = stID
    and buildDate = bDate;
delimiter ;
```
https://docs.google.com/presentation/d/1RvMDCe3Etc7kpidVLqKRRhn2vKf9_1d5W8Cf5H-jkqw/edit?usp=sharing
