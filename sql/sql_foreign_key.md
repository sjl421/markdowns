# sql_foreign_key

* 一个表中的 FOREIGN KEY 指向另一个表中的 PRIMARY KEY。
* ​

```sql
CREATE TABLE Orders(
  Id_O int NOT NULL,
  OrderNo int NOT NULL,
  Id_P int,
  PRIMARY KEY (Id_O),
  FOREIGN KEY (Id_P) REFERENCES Persons(Id_P)
)
```

```sql
CREATE TABLE Orders(
  Id_O int NOT NULL PRIMARY KEY,
  OrderNo int NOT NULL,
  Id_P int FOREIGN KEY REFERENCES Persons(Id_P)
)
```

```sql
CREATE TABLE Orders(
  Id_O int NOT NULL,
  OrderNo int NOT NULL,
  Id_P int,
  PRIMARY KEY (Id_O),
  CONSTRAINT fk_PerOrders FOREIGN KEY (Id_P)
  REFERENCES Persons(Id_P)
)
```

```sql
ALTER TABLE Orders ADD FOREIGN KEY (Id_P) REFERENCES Persons(Id_P);
```

```sql
ALTER TABLE Orders ADD CONSTRAINT fk_PerOrders FOREIGN KEY (Id_P) REFERENCES Persons(Id_P);
```

```sql
ALTER TABLE Orders DROP FOREIGN KEY fk_PerOrders;
```

```sql
ALTER TABLE Orders DROP CONSTRAINT fk_PerOrders;
```

