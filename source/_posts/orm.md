---
title: ORM框架
date: 2021-05-03 20:37:08
tags: ORM
---

## 一、ORM简介

`ORM`（对象关系映射，`Object relational mapping`，简称`ORM`）是一种为了解决业务中对象与关系数据库不匹配的现象而提出的框架技术，`ORM`
通过使用描述对象和数据库之间映射的元数据，将业务逻辑中的对象持久化到关系数据库中。

    持久化：持久化是将程序数据在持久状态和瞬时状态间转换的机制，瞬时数据持久化为持久数据。通俗易懂地讲就是，业务中处理的对象数据发生改变，

需要将其同步到数据库中保存，即`持久化`。（比如内存中的数据，是不能永久保存的，持久化至数据库中，能够长久保存）

实现持久化最基本的方式就是硬编码：为每一种增删改查的需求写相应的代码。可想而知，这种方式存在很多不足的地方：

- 缺乏弹性，一旦业务需求发生变更，就必须修改代码，扩展性以及可维护性差；

- 针对于每一种持久化需求，都需要写相应的代码，但很大一部分代码都是重复的；

针对于以上问题，`ORM`框架就被提出来。`ORM`作为业务处理对象与数据库连接的桥梁，通过对象的语法，完成关系型数据库的操作技术。

`ORM` 把数据库映射成对象。

- 数据库的表（`table`） --> 类（`class`）

- 记录（`record`，行数据）--> 对象（`object`）

- 字段（`field`）--> 对象的属性（`attribute`）

举个例子：

sql查询语句如下，

```sql
SELECT id, first_name, last_name, phone, birth_date, sex
FROM persons
WHERE id = 10
```

业务逻辑中操作数据库代码如下：

```ts
res = db.execSql(sql);
name = res[0]['FIRST_NAME'];
```

改用`ORM`框架之后：

```ts
let p = Person.get(10);
let name = p.first_name;
```

通过以上代码对比，`ORM`的优点就凸显出来了：业务开发者不需要去碰SQL语句，只需要使用`ORM`提供的接口，直接对对象进行操作，就可以实现与数据库的交互，不需要关心数据库底层的增删改查SQL语句。

## 二、ORM优缺点

`ORM`的优点：

- 数据模型都在一个地方定义，更容易更新和维护，也利于重用代码。

- `ORM` 有现成的工具，很多功能都可以自动完成，比如数据消毒、预处理、事务等等。

- 它迫使你使用 MVC 架构，`ORM` 就是天然的 Model，最终使代码更清晰。

- 基于 `ORM` 的业务代码比较简单，代码量少，语义性好，容易理解。

- 不必编写性能不佳的 SQL。

但是`ORM`也有相应的缺点：

- `ORM` 库不是轻量级工具，需要花很多精力学习和设置。

- 对于复杂的查询，`ORM` 要么是无法表达，要么是性能不如原生的 SQL。

- `ORM` 抽象掉了数据库层，开发者无法了解底层的数据库操作，也无法定制一些特殊的 SQL。

## 三、ORM框架

实用`ORM`框架：[sequelize](https://sequelize.org/v5/)

连接数据库：

```ts
const Sequelize = require('sequelize');
// option1:
sequelize = new Sequelize('database', 'username', 'password', {
    host: 'localhost',
    dialect: 'mysql' | 'mariadb' | 'postgres' | 'mssql',
});
// or
// option2:
const sequelize = new Sequelize('mysql://root:asd123@localhost:3306/mydb');

```

示例代码：

```ts
import {Sequelize, Model, DataTypes, BuildOptions} from 'sequelize';
import {
    HasManyGetAssociationsMixin,
    HasManyAddAssociationMixin,
    HasManyHasAssociationMixin,
    Association,
    HasManyCountAssociationsMixin,
    HasManyCreateAssociationMixin
} from 'sequelize';

// 连接数据库
const sequelize = new Sequelize('mysql://root:asd123@localhost:3306/mydb');

// 设置一个User表的操作对象
class User extends Model {
    public id!: number; // '!'非空断言必须得有
    public name!: string;
    public preferredName!: string | null; // for nullable fields
    
    // timestamps时间戳
    public readonly createdAt!: Date;
    public readonly updatedAt!: Date;
    
    // 设置关联表
    public getProjects!: HasManyGetAssociationsMixin<Project>; // Note the null assertions!
    public addProject!: HasManyAddAssociationMixin<Project, number>;
    public hasProject!: HasManyHasAssociationMixin<Project, number>;
    public countProjects!: HasManyCountAssociationsMixin;
    public createProject!: HasManyCreateAssociationMixin<Project>;
    
    public readonly projects?: Project[]; // Note this is optional since it's only populated when explicitly requested in code
    
    public static associations: {
        projects: Association<User, Project>;
    };
}

// 设置Project表
class Project extends Model {
    public id!: number;
    public ownerId!: number;
    public name!: string;
    
    public readonly createdAt!: Date;
    public readonly updatedAt!: Date;
}

// 设置Address表
class Address extends Model {
    public userId!: number;
    public address!: string;
    
    public readonly createdAt!: Date;
    public readonly updatedAt!: Date;
}

// 初始化Project表
Project.init({
    id: {
        type: DataTypes.INTEGER.UNSIGNED, // you can omit the `new` but this is discouraged
        autoIncrement: true,
        primaryKey: true,
    },
    ownerId: {
        type: DataTypes.INTEGER.UNSIGNED,
        allowNull: false,
    },
    name: {
        type: new DataTypes.STRING(128),
        allowNull: false,
    }
}, {
    sequelize,
    tableName: 'projects',
});

// 初始化User表
User.init({
    id: {
        type: DataTypes.INTEGER.UNSIGNED,
        autoIncrement: true,
        primaryKey: true,
    },
    name: {
        type: new DataTypes.STRING(128),
        allowNull: false,
    },
    preferredName: {
        type: new DataTypes.STRING(128),
        allowNull: true
    }
}, {
    tableName: 'users',
    sequelize: sequelize,
});

// 初始化Address表
Address.init({
    userId: {
        type: DataTypes.INTEGER.UNSIGNED,
    },
    address: {
        type: new DataTypes.STRING(128),
        allowNull: false,
    }
}, {
    tableName: 'address',
    sequelize: sequelize, // this bit is important
});

// 为Project表设置一个外键ownerId绑定约束到User表中
User.hasMany(Project, {
    sourceKey: 'id',
    foreignKey: 'ownerId',
    as: 'projects' // this determines the name in `associations`!
});

// 设置Address表的外键约束绑定到User表的id
Address.belongsTo(User, {targetKey: 'id'});
User.hasOne(Address, {sourceKey: 'id'});

// 测试
async function stuff() {
    const newUser = await User.create({
        name: 'Johnny',
        preferredName: 'John',
    });
    console.log(newUser.id, newUser.name, newUser.preferredName);

// 这个first Project记录生成时会与User的id绑定，ownerId对应User表里的id记录，而Project的id作为索引自增
    const project = await newUser.createProject({
        name: 'first!',
    });

// findByPk()，其中Pk为primary_key，即user表的索引ID，并且列出与该user有关的projects
    const ourUser = await User.findByPk(1, {
        include: [User.associations.projects],
        rejectOnEmpty: true, // Specifying true here removes `null` from the return type!
    });
    console.log(ourUser.projects![0].name);
}
```














