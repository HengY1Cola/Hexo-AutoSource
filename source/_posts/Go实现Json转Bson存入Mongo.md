---
title: Go实现Json转Bson存入Mongo
categories: 开发
keywords: Go操作Mongo
top_img: /img/4482734.png
cover: /img/4482740.webp
abbrlink: 9ef9f726
date: 2022-05-25 09:36:36
---

##  业务场景：

搞了个大作业，里面的话我们需要将前端传过来的Json直接存入到Mongo方便后面直接取出来分析。然后我看了很多Go语言操作MongoDB实现增删改查的文档，但是需要转为对应的结构体然后存入，但是我们这个Json的结构体没法固定，另辟蹊径吧，就有了以下操作（悄悄水一篇）

##  代码实现

以下的代码实现了**增与查**

```go
package common

import (
	"context"
	"encoding/json"
	"errors"
	"time"

	"go.mongodb.org/mongo-driver/bson/primitive"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
	"gopkg.in/mgo.v2/bson"
)

var (
	MongoPool *MongoDb // 对外暴露
)

type MongoDb struct {
	connection *mongo.Collection
}

func NewMongoDbPool() (*MongoDb, error) { // 创建实例化
	pool, err := ConnectToDB()
	if err != nil {
		return nil, err
	}
	return &MongoDb{
		connection: pool,
	}, nil
}

// ConnectToDB 与mongo创建连接
func ConnectToDB() (*mongo.Collection, error) {
	url := ""                      // mongo连接配置URL
	name := ""                     // 名字
	collection := ""               // 集合
	maxCollection := 10            // 最大连接数
	var timeout time.Duration = 10 // 设置10秒的超时时间
	ctx, cancel := context.WithTimeout(context.Background(), timeout)
	defer cancel()
	o := options.Client().ApplyURI(url)
	o.SetMaxPoolSize(uint64(maxCollection))
	client, err := mongo.Connect(ctx, o)
	if err != nil {
		return nil, err
	}
	return client.Database(name).Collection(collection), nil // 返回的连接直接对应的是集合
}

func (m *MongoDb) jsonStr2Bson(str string) (interface{}, error) {
	var want interface{}
	err := bson.UnmarshalJSON([]byte(str), &want)
	if err != nil {
		return nil, err
	}
	return want, nil
}

// InsertToDb 直接插入Json字段
func (m *MongoDb) InsertToDb(wantStr string) (string, error) {
	if wantStr == "" {
		return "", errors.New("转换的字符串为空")
	}
	want, err := m.jsonStr2Bson(wantStr)
	if err != nil {
		return "", err
	}
	res, err := m.connection.InsertOne(context.TODO(), want)
	if err != nil {
		return "", err
	}
	id, ok := res.InsertedID.(primitive.ObjectID)
	if !ok {
		return "", errors.New("断言错误")
	}
	return id.Hex(), nil
}

// FindInfoByField 通过字段与KEY进行查询
func (m *MongoDb) FindInfoByField(field, want string) (string, error) {
	ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
	defer cancel()
	filter := bson.M{field: want}
	cursor, err := m.connection.Find(ctx, filter)
	if err != nil {
		return "", err
	}
	defer cursor.Close(ctx)
	var temp []bson.M
	if err = cursor.All(context.Background(), &temp); err != nil {
		return "", err
	}
	if len(temp) == 0 {
		return "", nil
	}
	jsonInfo, err := json.Marshal(temp)
	if err != nil {
		return "", err
	}
	return string(jsonInfo), nil
}

// FindInfoById 通过Mongo自己的ID进行查询
func (m *MongoDb) FindInfoById(id string) (string, error) {
	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	defer cancel()
	objID, _ := primitive.ObjectIDFromHex(id)
	filter := bson.M{"_id": objID}
	cursor, err := m.connection.Find(ctx, filter)
	if err != nil {
		return "", err
	}
	defer cursor.Close(ctx)
	var temp []bson.M
	if err = cursor.All(context.Background(), &temp); err != nil {
		return "", err
	}
	if len(temp) == 0 {
		return "", nil
	}
	jsonInfo, err := json.Marshal(temp[0])
	if err != nil {
		return "", err
	}
	return string(jsonInfo), nil
}
```