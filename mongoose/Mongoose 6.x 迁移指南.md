原文: <https://mongoosejs.com/docs/migrating_to_6.html>

## 从 5.x 迁移到 6.x

在从 Mongoose 5.x 迁移到 Mongoose 6.x 时，有几个[反向破坏性变化](https://github.com/Automattic/mongoose/blob/master/CHANGELOG.md)需要注意。

如果你还在使用 Mongoose 4.x，请阅读[Mongoose 4.x to 5.x migration guide](/docs/migrating_to_5.html)并先升级到 Mongoose 5.x。

### 版本要求

Mongoose 现在需要 Node.js >= 12.0.0。Mongoose 仍然支持 3.0.0 以下的 MongoDB server 版本。

### MongoDB Driver 4.0

Mongoose 现在使用[MongoDB Node driver](https://www.npmjs.com/package/mongodb)的 4.x 版本。
请参阅[MongoDB Node Driver 的迁移指南](https://github.com/mongodb/node-mongodb-native/blob/4.0/docs/CHANGES_4.0.0.md)以了解详细的信息。以下是一些最值得注意的变化:

- MongoDB Driver 4.x 是用 TypeScript 写的，并且有自己的 TypeScript 类型定义。这些可能与`@types/mongodb`冲突，所以如果你有 TypeScript 编译器错误，请确保你升级到[最新版本的`@types/mongodb`](https://www.npmjs.com/package/@types/mongodb)，它是一个空 stub。
- connections 的`poolSize`选项已经被[替换为`minPoolSize`和`maxPoolSize`](https://github.com/mongodb/node-mongodb-native/blob/4.1/docs/CHANGES_4.0.0.md#connection-pool-options)。Mongoose 5.x 的`poolSize`选项等同于 Mongoose 6 的`maxPoolSize`选项。`maxPoolSize`的默认值已经增加到 100。
- `updateOne()`和`updateMany()`的结果现在不同。
- `deleteOne()`和`deleteMany()`的结果不再有`n`属性。

```javascript
let res = await TestModel.updateMany({}, { someProperty: "someValue" });

res.matchedCount; // 找到的符合过滤器的documents数，取代`res.n`
res.modifiedCount; // 修改的documents数，取代`res.nModified`
res.upsertedCount; // 插入的documents数，取代`res.upserted`
```

```javascript
let res = await TestModel.deleteMany({});

// 在 Mongoose 6 中为: `{ acknowledged: true, deletedCount: 2 }`
// 在 Mongoose 5 中为: `{ n: 2, ok: 1, deletedCount: 2 }`
res;

res.deletedCount; // 被删除的documents数，取代`res.n`
```

### 不再有废弃选项警告

选项`useNewUrlParser`, `useUnifiedTopology`, `useFindAndModify`, 和`useCreateIndex`不再支持。Mongoose 6 中`useNewUrlParser`、`useUnifiedTopology`和`useCreateIndex`默认为`true`，而`useFindAndModify`为`false`。请从你的代码中删除这些选项:

```javascript
// 不再需要:
mongoose.set("useFindAndModify", false);

await mongoose.connect("mongodb://localhost:27017/test", {
  useNewUrlParser: true, // <-- 不再需要
  useUnifiedTopology: true, // <-- 不再需要
});
```

### 连接时的`asPromise()`方法

Mongoose 连接不再是[thenable](https://masteringjs.io/tutorials/fundamentals/thenable)。这意味着`await mongoose.createConnection(uri)` **不再等待 Mongoose 连接**。使用`mongoose.createConnection(uri).asPromise()`代替。参见[#8810](https://github.com/Automattic/mongoose/issues/8810)。

```javascript
// 以下代码在Mongoose 6中已不再适用
await mongoose.createConnection(uri);

// 改为
await mongoose.createConnection(uri).asPromise();
```

### `mongoose.connect()`返回一个 Promise

`mongoose.connect()`函数现在总是返回一个 Promise，**而不是**一个 Mongoose 实例。

### 重复查询的执行

Mongoose 不再允许重复执行同一个查询对象。如果你这样做，你会得到一个 `Query was already executed` 的错误。两次执行同一个查询实例通常表示混合了回调和 Promise，但是如果你需要两次执行同一个查询，你可以调用`Query#clone()`来拷贝一个查询并重新执行它。见[gh-7398](https://github.com/Automattic/mongoose/issues/7398)

```javascript
// 结果是 `Query was already executed` 的错误，因为从技术上讲，这个`find()`查询执行了两次
await Model.find({}, function (err, result) {});

const q = Model.find();
await q;
await q.clone(); // 可以`clone()`查询，以便再次执行查询
```

### `strictQuery`现在默认等于`strict`

~~Mongoose 不再支持`strictQuery`选项。你现在必须使用`strict`.~~ 从 Mongoose 6.0.10 开始，我们重新启用了`strictQuery`选项。 然而，`strictQuery`默认与`strict`绑定。这意味着，默认情况下，Mongoose 会过滤掉不在 schema 中的查询过滤属性。

```javascript
const userSchema = new Schema({ name: String });
const User = mongoose.model("User", userSchema);

// 默认情况下，这等同于`User.find()`，因为Mongoose会过滤掉`notInSchema`
await User.find({ notInSchema: 1 });

// 设置`strictQuery: false`以选择不过滤
await User.find({ notInSchema: 1 }, null, { strictQuery: false });
// 等同于:
await User.find({ notInSchema: 1 }).setOptions({ strictQuery: false });
```

你也可以全局禁用`strictQuery`来覆盖默认配置:

```javascript
mongoose.set("strictQuery", false);
```

### MongoError 现在改为 MongoServerError

在 MongoDB Node.js Driver v4.x 中，'MongoError' 现在改为 'MongoServerError'。请修改任何依赖于硬编码字符串'MongoError'的代码。

### 默认拷贝判别器 Schema

Mongoose 现在默认会拷贝判别器 schema。这意味着如果你使用递归嵌入式判别器，你需要将`{ clone: false }`传递给`discriminator()`。

```javascript
// 在Mongoose 6中，这两者是等价的:
User.discriminator("author", authorSchema);
User.discriminator("author", authorSchema.clone());

// 如果`clone()`导致问题，请传递`clone: false`
User.discriminator("author", authorSchema, { clone: false });
```

### schema 定义的 document 键顺序

Mongoose 现在按照 schema 中指定的键的顺序来保存对象，而不是按照用户定义的对象。所以`Object.key(new User({ name: String, email: String }).toObject()`是`['name', 'email']`还是`['email', 'name']`取决于`name`和`email`在你 schema 中的定义顺序。

```javascript
const schema = new Schema({
  profile: {
    name: {
      first: String,
      last: String,
    },
  },
});
const Test = db.model("Test", schema);

const doc = new Test({
  profile: { name: { last: "Musashi", first: "Miyamoto" } },
});

// 请注意，'first'在'last'之前，尽管 `new Test()`的参数翻转了键的顺序
// Mongoose使用schema的键序，而不是object提供的键序
assert.deepEqual(Object.keys(doc.toObject().profile.name), ["first", "last"]);
```

### `sanitizeFilter`和`trusted()`.

Mongoose 6 在全局和查询中引入了一个新的`sanitizeFilter`选项，可以防御[查询选择器注入攻击](https://thecodebarbarian.com/2014/09/04/defending-against-query-selector-injection-attacks.html)。如果你启用了`sanitizeFilter`，Mongoose 将把查询过滤器中的任何对象包装到`$eq`中。

```javascript
// Mongoose将把这个过滤器转换成`{ username: 'val', pwd: { $eq: { $ne: null } } }`，防止查询选择器的注入
await Test.find({ username: "val", pwd: { $ne: null } }).setOptions({
  sanitizeFilter: true,
});
```

为了显式地允许查询选择器, 请用`mongoose.trusted()`:

```javascript
await Test.find({
  username: "val",
  pwd: mongoose.trusted({ $ne: null }),
}).setOptions({ sanitizeFilter: true });
```

### 删除了`omitUndefined`

在 Mongoose 5.x 中，在更新操作中把一个键设置为`undefined`等同于把它设置为`null`。

```javascript
const res = await Test.findOneAndUpdate(
  {},
  { $set: { name: undefined } },
  { new: true }
);

res.name; // null
```

Mongoose 5.x 支持一个`omitUndefined`选项，以去除值为`undefined`的键。
在 Mongoose 6.x 中，`omitUndefined`选项已被删除，Mongoose 将始终去除出未定义的键。

```javascript
// 在Mongoose 6中，相当于`findOneAndUpdate({}, {}, { new: true })`，因为Mongoose会去除`name: undefined`
const res = await Test.findOneAndUpdate(
  {},
  { $set: { name: undefined } },
  { new: true }
);
```

### 默认函数的 Document 参数

Mongoose 现在将 document 作为第一个参数传递给`default`函数，这对于使用带有 defaults 的[箭头函数](https://masteringjs.io/tutorials/fundamentals/arrow)很有帮助。

如果你期望一个有不同参数的函数传递给`default`，例如`default: mongoose.Types.ObjectId`，这可能会影响你。参见[gh-9633](https://github.com/Automattic/mongoose/issues/9633)。如果你要传递一个不利用 document 的默认函数，将`default: myFunction`改为`default: () => myFunction()`以避免意外地传递可能改变行为的参数。

```javascript
const schema = new Schema({
  name: String,
  age: Number,
  canVote: {
    type: Boolean,
    // default函数现在可以接收一个`doc`参数，对箭头函数有帮助
    default: (doc) => doc.age >= 18,
  },
});
```

### 启用数组代理

Mongoose 数组现在是 ES6 Proxy。你不再需要在直接修改数组后调用`markModified()`。

```javascript
const post = await BlogPost.findOne();

post.tags[0] = "javascript";
await post.save(); // 变更会生效, 不需要`markModified()`!
```

### `typePojoToMixed`

在 Mongoose 6 中，用`type: { name: String }`声明的 schema 路径成为单个嵌套的 subdocument，而在 Mongoose 5 中是 Mixed 的。这消除了对`typePojoToMixed`选项的需要。参见[gh-7181](https://github.com/Automattic/mongoose/issues/7181)。

```javascript
// 在Mongoose 6中，以下代码将`foo`变成一个带有`name`属性的subdocument
// 在Mongoose 5中，以下代码会使`foo`成为`Mixed`类型，除非你设置`typePojoToMixed: true`
const schema = new Schema({
  foo: { type: { name: String } },
});
```

### `strictPopulate()`

如果你`populate()`的路径未在你的 schema 中定义，Mongoose 现在会抛出一个错误。这只适用于我们可以推断出本地 schema 的情况，比如你使用`Query#populate()`，而不是你在 POJO 上调用`Model.populate()`时。参见[gh-5124](https://github.com/Automattic/mongoose/issues/5124)。

### Subdocument `ref` 函数上下文

当用函数`ref`或`refPath`Populate 一个 subdocument 时，`this`现在是被 Populate 的 subdocument，而不是顶层 document。参见[#8469](https://github.com/Automattic/mongoose/issues/8469)。

```javascript
const schema = new Schema({
  works: [
    {
      modelId: String,
      data: {
        type: mongoose.ObjectId,
        ref: function (doc) {
          // 在Mongoose 6中，`doc`是数组元素，所以你可以访问`modelId`
          // 在Mongoose 5中，`doc`是顶层document
          return doc.modelId;
        },
      },
    },
  ],
});
```

### schema 保留名称警告

现在使用`save`, `isNew`, 和其它 Mongoose 保留字作为 schema 路径名时会触发一个警告，而不是一个错误。你可以通过在 schema 选项中设置`supressReservedKeysWarning`来抑制这个警告:`new Schema({ save: String }, { supressReservedKeysWarning: true })`。请注意，在使用那些依赖这些保留字的插件时可能会出现问题。

### subdocument 路径

单个嵌套的 subdocument 已经被重新命名为 "subdocument paths"。所以`SchemaSingleNestedOptions`现在为`SchemaSubdocumentOptions`，`mongoose.Schema.Types.Embedded`现在为`mongoose.Schema.Types.Subdocument`。参见[gh-10419](https://github.com/Automattic/mongoose/issues/10419)

### 创建聚合游标

`Aggregate#cursor()`现在返回一个 AggregationCursor 实例，以便与`Query#cursor()`一致。你不再需要通过`Model.aggregate(pipeline).cursor().exec()`来获得一个聚合游标，只需要`Model.aggregate(pipeline).cursor()`。

### `autoCreate`默认为`true`

`autoCreate`默认为`true`，除非 readPreference 是 secondary 或 secondaryPreferred，这意味着 Mongoose 将尝试在创建索引前创建每个 model 的底层 collection。如果 readPreference 是 secondary 或 secondaryPreferred，Mongoose 将默认`autoCreate`和`autoIndex`为`false`，因为当连接到 secondary 时，`createCollection()`和`createIndex()`都会失败。

### 不再有 `context: 'query'`

查询的`context`选项已经被移除。现在 Mongoose 总是使用`context = 'query'`。

### 自定义验证器与 Populate 路径

Mongoose 6 总是用未 Populate 的路径调用验证器(也就是用 id 而不是 document 本身)。在 Mongoose 5 中，如果路径是 Populate 的，Mongoose 会用 Populate 的 document 调用验证器。参见[#8042](https://github.com/Automattic/mongoose/issues/8042)

### 复制集的断开连接事件

当连接到一个副本集时，如果与主副本的连接丢失时，连接现在会发出 'disconnected'。在 Mongoose 5 中，连接只在与复制集的所有成员失去连接时发出 'disconnected'。

然而，Mongoose 6 不会在连接断开时缓冲命令。因此，你仍然可以成功地执行命令，比如用`readPreference = 'secondary'`查询，即使 Mongoose 连接处于断开状态。

### 删除`execPopulate()`

`Document#populate()`现在返回一个 Promise 并且无法链式调用 populate。

- 用`await doc.populate(['path1', 'path2']`代替`await doc.populate('path1').populate('path2') execPopulate();`。
- 用`await doc.populate([{path: 'path1', select: 'select1'}, {path: 'path2', select: 'select2'}])`代替`await doc.populate('path1', 'select1').populate('path2', 'select2').execPopulate();`

### `create()`使用空数组

`await Model.create([])`在 v6.0 中会返回一个空数组，在 v5.0 中，它返回`undefined`。如果你的任何代码正在检查输出是否是`undefined`，你需要修改它并且认定`await Model.create(...)`在提供数组时总是返回一个数组。

### 删除了嵌套路径的合并

`doc.set({ child: { age: 21 })`现在无论`child`是一个嵌套路径还是一个 subdocument，其作用都是一样的。Mongoose 将覆盖`child`的值。在 Mongoose 5 中，如果`child`是一个嵌套路径，这个操作将合并`child`。

### ObjectId `valueOf()`

Mongoose 现在为 ObjectIds 增加了一个`valueOf()`函数。这意味着你现在可以使用`==`来比较一个 ObjectId 和一个字符串。

```javascript
const a = ObjectId("6143b55ac9a762738b15d4f0");

a == "6143b55ac9a762738b15d4f0"; // true
```

### 不可变的`createdAt`

如果你设置了`timestamps: true`，Mongoose 现在会使`createdAt`属性变得`immutable`。见[gh-10139](https://github.com/Automattic/mongoose/issues/10139)

### 移除验证器 `isAsync`

`isAsync`不再是`validate`的一个选项。使用`异步函数`代替。

### 删除了 `safe`

`safe`不再是 schema、查询或`save()`的一个选项。使用`writeConcern`代替。

### schema 类型 `set` 参数

Mongoose 现在使用`priorValue`作为第二个参数来调用 setter 函数，而不是 Mongoose 5 中的`schemaType`。

```javascript
const userSchema = new Schema({
  name: {
    type: String,
    trimStart: true,
    set: trimStartSetter,
  },
});

// 在v5.x中，参数是(value, schemaType)，在v6.x中，参数是(value, priorValue, schemaType)
function trimStartSetter(val, priorValue, schemaType) {
  if (schemaType.options.trimStart && typeof val === "string") {
    return val.trimStart();
  }
  return val;
}

const User = mongoose.model("User", userSchema);

const user = new User({ name: "Robert Martin" });
console.log(user.name); // 'robert martin'
```

### `toObject()`和`toJSON()`使用嵌套 schema `minimize`

这个变化在技术上是与 5.10.5 一起发布的，但是[对从 5.9.x 迁移到 6.x 的用户造成了问题](https://github.com/Automattic/mongoose/issues/10827)。
在 Mongoose `< 5.10.5`中，`toObject()`和`toJSON()`将默认使用顶层 schema 的`minimize`选项。

```javascript
const child = new Schema({ thing: Schema.Types.Mixed });
const parent = new Schema({ child }, { minimize: false });
const Parent = model("Parent", parent);
const p = new Parent({ child: { thing: {} } });

// 在v5.10.4中，将包含`child.thing`，因为`toObject()`使用`parent` schema 的`minimize`选项。
// 在`>= 5.10.5`中，`child.thing`被省略了，因为`child` schema 有`minimize: true`。
console.log(p.toObject());
```

作为一种变通方法，你可以明确地将`minimize`传递给`toObject()`或`toJSON()`:

```javascript
console.log(p.toObject({ minimize: false }));
```

或者内联定义 `child` schema(仅 Mongoose 6)以继承父 schema 的`minimize`选项。

```javascript
const parent = new Schema(
  {
    // 在顶层 schema 中设置 'minimize' 选项，内联的schema将隐式的带有'minimize'
    child: { type: { thing: Schema.Types.Mixed } },
  },
  { minimize: false }
);
```

## TypeScript 的变化

`Schema`类现在需要 3 个泛型参数，而不是 4 个。第 3 个泛型参数`SchemaDefinitionType`现在与第 1 个泛型参数`DocType`相同。将`new Schema<UserDocument, UserModel, User>(schemaDefinition)`替换为`new Schema<UserDocument, UserModel>(schemaDefinition)`。

`Types.ObjectId`现在是一个类，这意味着你在使用`new mongoose.Types.ObjectId()`创建一个新的 ObjectId 时不能再省略`new`。
目前，你仍然可以在 JavaScript 中省略`new`，但你必须在 TypeScript 中使用`new`。

以下的遗留类型已经被移除:

- `ModelUpdateOptions`
- `DocumentQuery`
- `HookSyncCallback`
- `HookAsyncCallback`
- `HookErrorCallback`
- `HookNextFunctio`
- `HookDoneFunction`
- `SchemaTypeOpts`
- `ConnectionOptions`
