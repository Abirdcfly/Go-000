# 作业
## 题目
我们在数据库操作的时候，比如 `dao` 层中当遇到一个 `sql.ErrNoRows` 的时候，是否应该 `Wrap` 这个 `error`，抛给上层。为什么？应该怎么做请写出代码

## 解答
取决于代码内容。

比如这个函数是修改（editXXX）某个内容，找不到ID对应的内容大概率需要向上层抛出错误。但是如果函数是查询（Findxxx），那么找不到某个ID对应的内容很可能是预期的情况之一。甚至可能会是更新或新建函数（UpdateOrCreateXXX）那么这个函数就不应该抛出错误。

## 代码实现：
```go
// DAO 层

var (
	ErrRecordNotFound = errors.New("record not found")
	......
)

func (m *Model) Get(ID string) (record *model.Record, err error) {
	err = db.Table(m.TableName()).Where("id = ?", ID).First(&record).Error
	if errors.Is(err, sql.ErrNoRows) {
		err = ErrRecordNotFound
    return
	}
	if err != nil {
		err = errors.Wrap(err, fmt.Sprintf("Get ID:[%s]", ID))
	}
	return
}

func (m *Model) UpdateOrCreate(record *model.Record) (err error) {
  var old *model.Record
	err = db.Table(m.TableName()).Where("id = ?", record.ID).First(&old).Error
	if !errors.Is(err, sql.ErrNoRows) {
    return errors.Wrap(err, fmt.Sprintf("UpdateOrCreate ID:[%s]", record.ID))
	}
  err = db.Table(m.TableName()).Save(record)
	return err
}
```
