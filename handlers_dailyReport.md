## accessRollHelper.js
这是修改之后的departmentSearcher和contentIdsSearcher
```javascript
departmentSearcher = function (waterfallCallback) {
    models.get(req.session.lastDb, 'Department', DepartmentSchema).aggregate(
        {
            $unwind: '$users'
        },
        {
            $match: {
                //因为这里users是个数组，很明显$match不满足
                //如果要用$in也不行，$in查询的条件是数组
                //所以这里直接用$unwind直接把数组拆开了
                users: objectId(req.session.uId)
            }
        }, {
            $project: {
                _id: 1
            }
        },
        waterfallCallback);
};
contentIdsSearcher = function (deps, waterfallCallback) {
    var everyOne = rewriteAccess.everyOne();
    var owner = rewriteAccess.owner(req.session.uId);
    var group = rewriteAccess.group(req.session.uId, deps);
    var whoCanRw = [everyOne, owner, group];
    var matchQuery = {
        $or: whoCanRw
    };
    Model.aggregate(
        {
            //这里同上面那个是一样的问题，而且要注意的是当groups.users数组内为空时也是要有一条数据的
            //所以加上了preserveNullAndEmptyArrays: true
            $unwind: {
                path: '$groups.users',
                preserveNullAndEmptyArrays: true
            }
        },
        {
            $unwind: {
                path: '$groups.group',
                preserveNullAndEmptyArrays: true
            }
        },
        {
            $match: matchQuery
        },
        {
            $project: {
                _id: 1
            }
        },
        waterfallCallback
    );
};
```
这里是一个异步操作
waterfall(tasks, [callback]) （多个函数依次执行，且前一个的输出为后一个的输入）
```javascript
waterfallTasks = [departmentSearcher, contentIdsSearcher];
async.waterfall(waterfallTasks, function (err, result) {
    if (err) {
        return waterfallCb(err);
    }
    resultArray = _.pluck(result, '_id');
    waterfallCb(null, resultArray);
});
```
## dailyReport.js（[具体代码]()）
**引入accessRollHelper模块**
```javascript
var accessRoll = require('../helpers/accessRollHelper')(models);
```
由于删除和修改的方法与之前无异，所以不过多赘述
主要分析插入函数以及查询函数：

### create
在我的理解中，既然登录用户user的relatedEmployee对应一个Employees中的Employee，那么通过user创建的dailyReport权限应该和它对应的Employee对应
所以在创建dailyReport之前，先通过`uId`查到`user`，再用`user.relatedEmployee`查找到Employee，然后把Employee的`whoCanRW`和`groups`赋值给要创建的dailyReport
```javascript
data.userId = req.session.uId;
var date = new Date();
var nowDay = date.getFullYear() + "-" + (date.getMonth() + 1) + "-" + date.getDate();
data.dateStr = nowDay;
data.status = 'new';
userModel.findOne({_id : req.session.uId}, function (err, user) {
    if (err) {
        return next(err);
    }
    employeeModel.findOne({_id: user.relatedEmployee}, function (err, employee) {
        if (err) {
            return next(err);
        }
        data.whoCanRW = employee.whoCanRW;
        data.groups= {};
        data.groups.owner = employee.groups.owner;
        data.groups.users = employee.groups.users;
        data.groups.group = employee.groups.group;
        dailyReport = new dailyReportModel(data);//到这里就获得了可操作的类似于collection的对象
        dailyReport.save(function (err, result) {
            if (err) {
                return next(err);
            }
            res.status(200).send(result);
        });
    });
});
```

### getDailyReport
这里很明显使用dailyReportId查到对应的日报，这里我还要获取用户的姓名，所以用了$lookup查到了employee
```javascript
Reports
    .aggregate([
        {
            $match:{_id : id}
        },
        {
            $lookup:
                {
                    from: 'Users',
                    localField: 'userId',
                    foreignField: '_id',
                    as: 'user'
                }
        },
        {
            //上一个语句执行完返回的是一个数组，我需要的是一条数据，所以用$unwind直接把数组拆了
            $unwind: '$user'
        },
        {
            $lookup:
                {
                    from: 'Employees',
                    localField: 'user.relatedEmployee',
                    foreignField: '_id',
                    as: 'employee'
                }
        },
        {
            $project: {
                _id     :1,
                userId  : 1,
                dateStr : 1,
                status  : 1,
                content : 1,
                review  : 1,
                groups  : 1,
                whoCanRW: 1,
                //这里由于不需要其他操作，所以直接用$arrayElemAt获取数组中的第一个值
                employee: {$arrayElemAt:['$employee', 0]},
                user    :'$user'
            }
        }
    ])
    ......
```

### getList

这里加载了accessRollHelper这个模块方法
```javascript
var accessRollSearcher = function(cb) {
    accessRoll(req, dailyReportModel, cb);
};
```
accessRollSearcher在下面的异步方法中执行，结果存在`result[1]`中
```javascript
parallelTasks = [getTotal, accessRollSearcher];

//并行且无关联，有一个流程出错就抛错
async.parallel(parallelTasks, function (err, result) {
    var count;
    var response = {};

    if (err) {
        return next(err);
    }

    count = result[0] || 0;
    var dailyReportIds = result[1].objectID();

    ......
});
```
