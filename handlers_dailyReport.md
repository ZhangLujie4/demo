
# dailyReport.js
**引入accessRollHelper模块**
```javascript
var accessRoll = require('../helpers/accessRollHelper')(models);
```
由于删除和修改的方法与之前无异，所以不过多赘述
主要分析插入函数以及查询函数：

## create
```javascript
//插入数据
this.create = function (req, res, next) {
    var db = req.session.lastDb;
    var dailyReportModel = models.get(db, 'dailyReport', dailyReportSchema);
    var employeeModel = models.get(db, 'Employees', employeeSchema);
    var userModel = models.get(db, 'Users', userSchema);
    var dailyReport;
    var data = req.body;//获得文档类型
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
            console.log(data);
            dailyReport = new dailyReportModel(data);//到这里就获得了可操作的类似于collection的对象

            dailyReport.save(function (err, result) {
                if (err) {
                    return next(err);
                }
                res.status(200).send(result);
            });
        });
    });
};
```
## getList
```javascript
//查询所有dailyReportModel类型数据
this.getList = function (req, res, next) {
    var dailyReportModel = models.get(req.session.lastDb, 'dailyReport', dailyReportSchema);
    var employeeModel = models.get(req.session.lastDb, 'Employees', employeeSchema);
    var data = req.query;
    var sort = {status : 1, _id : -1};
    var paginationObject = pageHelper(data);
    var limit = paginationObject.limit;
    var skip = paginationObject.skip;
    var parallelTasks;

    var accessRollSearcher = function(cb) {
        accessRoll(req, dailyReportModel, cb);
    };


    //返回共有多少条数据
    var getTotal = function (pCb) {

        dailyReportModel.count(function (err, _res) {
            if (err) {
                return pCb(err);
            }

            pCb(null, _res);
        });
    };

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

        dailyReportModel
            .aggregate([
                {
                    $match:{'_id': {$in : dailyReportIds}}
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
                        _id     : 1,
                        userId  : 1,
                        dateStr : 1,
                        status  : 1,
                        content : 1,
                        review  : 1,
                        groups  : 1,
                        whoCanRW: 1,
                        employee: {$arrayElemAt:['$employee', 0]},
                        user    :'$user'
                    }
                }
            ])
            .skip(skip)
            .limit(limit)
            .sort(sort)
            .exec(function (err, dailyReports) {
                if (err) {
                    return next(err);
                }

                response.total = count;
                response.data = dailyReports;
                res.status(200).send(response);
            });
    });

};
```

## getDailyReport
```javascript
//找到当前用户日报
this.getDailyReport = function (req, res, next) {
    var Reports = models.get(req.session.lastDb, 'dailyReport', dailyReportSchema);
    var id = mongoose.Types.ObjectId(req.params.id);

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
                    employee: {$arrayElemAt:['$employee', 0]},
                    user    :'$user'
                }
            }
        ])
        .exec(function (err, dailyReports) {
            if (err) {
                return next(err);
            }

            res.status(200).send(dailyReports[0]);
        });
};
```
