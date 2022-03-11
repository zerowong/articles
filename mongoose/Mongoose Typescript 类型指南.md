原文: <https://mongoosejs.com/docs/typescript.html>

# TypeScript 支持

Mongoose 在 v5.11.0 中引入了[正式 TypeScript 支持 ](https://thecodebarbarian.com/working-with-mongoose-in-typescript.html)。
Mongoose 的`index.d.ts`文件支持各种各样的语法，并尽可能的与`@types/mongoose`兼容。
本指南描述了 Mongoose 推荐的在 TypeScript 中使用 Mongoose 的方法。

### 创建你的第一个 document

要在 TypeScript 中开始使用 Mongoose，你需要:

1. 创建一个代表 MongoDB 中的 document 的 interface
2. 创建一个与 document interface 相对应的[Schema](/docs/guide.html)
3. 创建一个 model
4. [连接到 MongoDB](/docs/connections.html)

```typescript
import { Schema, model, connect } from "mongoose";

// 1. 创建一个代表 MongoDB 中的 document 的 interface
interface User {
  name: string;
  email: string;
  avatar?: string;
}

// 2. 创建一个与 document interface 相对应的 Schema
const schema = new Schema<User>({
  name: { type: String, required: true },
  email: { type: String, required: true },
  avatar: String,
});

// 3. 创建一个 model
const UserModel = model<User>("User", schema);

run().catch((err) => console.log(err));

async function run(): Promise<void> {
  // 4. 连接到 MongoDB
  await connect("mongodb://localhost:27017/test");

  const doc = new UserModel({
    name: "Bill",
    email: "bill@initech.com",
    avatar: "https://i.imgur.com/dM7Thhn.png",
  });
  await doc.save();

  console.log(doc.email); // 'bill@initech.com'
}
```

作为开发者，你有责任确保你的 document interface 与你的 Mongoose schema 保持一致。
例如，如果 `email` 在你的 Mongoose schema 中是 `required`，但在你的 document interface 中是 option 的，Mongoose 不会报错。

`UserModel()`构造函数返回一个`HydratedDocument<User>`的实例。
`User`是一个*document interface*，它代表了 MongoDB 中`User`对象的原始对象结构。
`HydratedDocument<User>`代表了一个水合的 Mongoose document，具有 method、virtuals 和其他 Mongoose 特定的功能。

```typescript
import { HydratedDocument } from "mongoose";

const doc: HydratedDocument<User> = new UserModel({
  name: "Bill",
  email: "bill@initech.com",
  avatar: "https://i.imgur.com/dM7Thhn.png",
});
```

### ObjectIds 和其他 Mongoose 类型

要定义一个`ObjectId`类型的属性，你应该在 TypeScript document interface 中使用`Types.ObjectId`。
你应该在你的 schema 定义中使用`'ObjectId`或`Schema.Types.ObjectId`。

```typescript
import { Schema, Types } from "mongoose";

// 1. 创建一个代表 MongoDB 中的 document 的 interface
interface User {
  name: string;
  email: string;
  // 使用 `Types.ObjectId`
  organization: Types.ObjectId;
}

// 2. 创建一个与 document interface 相对应的 Schema
const schema = new Schema<User>({
  name: { type: String, required: true },
  email: { type: String, required: true },
  // 和 `Schema.Types.ObjectId` 类型定义
  organization: { type: Schema.Types.ObjectId, ref: "Organization" },
});
```

这是因为`Schema.Types.ObjectId`是一个[继承自 SchemaType 的类](/docs/schematypes.html)，不是你用来创建一个新的 MongoDB ObjectId 的类。

### 使用`extends Document`

另外，你的 document interface 可以扩展 Mongoose 的`Document`类。
许多 Mongoose TypeScript 代码库使用以下方法。

```typescript
import { Document, Schema, model, connect } from "mongoose";

interface User extends Document {
  name: string;
  email: string;
  avatar?: string;
}
```

这种方法是可行的，但我们建议你的 document interface 不要扩展`Document`。
使用`extends Document`使 Mongoose 难以推断哪些属性是否存在于[query filters](/docs/queries.html)、[lean documents](/docs/tutorials/lean.html)等。

我们建议你的 document interface 包含你的 schema 中定义的属性，并与 MongoDB 中的 document 一致。
尽管你可以将[实例方法](/docs/guide.html#methods)添加到你的 document interface 中，但我们并不推荐这样做。

### 使用自定义类型定义

如果 Mongoose 内置的`index.d.ts`文件无法使用，你可以在你的`package.json`中写脚本中删除它，如下图。
然而，在你这样做之前，请[在 Mongoose 的 GitHub 页面上开一个 issue](https://github.com/Automattic/mongoose/issues/new)并描述你遇到的问题。

```json
{
  "postinstall": "rm ./node_modules/mongoose/index.d.ts"
}
```

# Schema

Mongoose [schemas](/docs/guide.html) 用来告诉 Mongoose 你的 document 的结构。
Mongoose schema 与 TypeScript interface 是分开的，所以你需要同时定义 _document interface_ 和 _schema_。

```typescript
import { Schema } from "mongoose";

interface User {
  name: string;
  email: string;
  avatar?: string;
}

const schema = new Schema<User>({
  name: { type: String, required: true },
  email: { type: String, required: true },
  avatar: String,
});
```

默认情况下，Mongoose 不会检查你的 document interface 是否与你的 schema 一致。
如上面的代码所示，`email`在 document interface 中是可选的，而在`schema`中是`required`，但这不会报错。

## 泛型参数

TypeScript 中的 Mongoose `Schema` 类有 3 个[泛型参数](https://www.typescriptlang.org/docs/handbook/2/generics.html)。

- `DocType` - 一个描述数据在 MongoDB 中如何保存的 interface
- `M` - Mongoose model 类型。如果没有要定义的查询辅助方法或实例方法，可以省略。
  - 默认:`Model<DocType, any, any>`。
- `TInstanceMethods` - 一个包含 schema 实例方法的 interface。
  - 默认: `{}`

类型定义:

```typescript
class Schema<
  DocType = any,
  M = Model<DocType, any, any>,
  TInstanceMethods = {}
> extends events.EventEmitter {
  // ...
}
```

第一个泛型参数`DocType`表示 Mongoose 如何将 document 存储到 MongoDB 中。
Mongoose 将`DocType`包装在 Mongoose document 中，可用于类似在 document 中间件中表示 `this`。
例如:

```typescript
schema.pre("save", function (): void {
  console.log(this.name); // TypeScript 能推断出 `this` 为 `mongoose.Document & User`
});
```

第二个泛型参数 `M` 是与 schema 一起使用的 model。Mongoose 在 schema 中定义的 model 中间件中使用 `M` 类型。

第三个泛型参数，`TInstanceMethods`用于为 schema 中定义的实例方法添加类型。

## Schema 与 interface 字段

Mongoose 的类型检查会确保 schema 中的每个路径都在 document interface 中定义。

例如，下面的代码会编译报错，因为`emaill`是 schema 中的一个路径，但没有在 `DocType`(即`User`) interface 中定义:

```typescript
import { Schema, Model } from "mongoose";

interface User {
  name: string;
  email: string;
  avatar?: string;
}

// TS:
// Object literal may only specify known properties, but 'emaill' does not exist in type ...
// Did you mean to write 'email'?
const schema = new Schema<User>({
  name: { type: String, required: true },
  emaill: { type: String, required: true },
  avatar: String,
});
```

然而，Mongoose 并不检查那些存在于 document interface 但不存在于 schema 中的路径。
例如，下面的代码不会报错:

```typescript
import { Schema, Model } from "mongoose";

interface User {
  name: string;
  email: string;
  avatar?: string;
  createdAt: number;
}

const schema = new Schema<User, Model<User>>({
  name: { type: String, required: true },
  email: { type: String, required: true },
  avatar: String,
});
```

这是因为 Mongoose 有许多方法为你的 schema 添加路径，这些路径应该包含在 `DocType` interface 中，但不需要你明确将这些路径放在`Schema()`构造函数中。例如[timestamps](https://masteringjs.io/tutorials/mongoose/timestamps)和[plugins](/docs/plugins.html)。

## 数组

当你在 document interface 中定义一个数组时，我们建议使用 Mongoose 的`Types.Array`类型来表示原始数组，或者使用`Types.DocumentArray`来表示 document 数组。

```typescript
import { Schema, Model, Types } from "mongoose";

interface BlogPost {
  _id: Types.ObjectId;
  title: string;
}

interface User {
  tags: Types.Array<string>;
  blogPosts: Types.DocumentArray<BlogPost>;
}

const schema = new Schema<User, Model<User>>({
  tags: [String],
  blogPosts: [{ title: String }],
});
```

在处理默认值时，使用`Types.DocumentArray`很有帮助。例如，`BlogPost`有一个`_id`属性，Mongoose 会默认设置。
如果你在上述情况下使用`Types.DocumentArray`，你将能够`push()`一个没有`_id`的 subdocument。

```typescript
const user = new User({ blogPosts: [] });

user.blogPosts.push({ title: "test" }); // 如果你定义为 `blogPosts: BlogPost[]` 则会报错
```

# Statics

Mongoose [models](/docs/models.html)没有明确的[static](/docs/guide.html#statics)泛型参数。
如果你的 model 需要静态方法，我们建议创建一个 [extends](https://www.typescriptlang.org/docs/handbook/interfaces.html) Mongoose 的 `Model` interface，如下所示:

```typescript
import { Model, Schema, model } from "mongoose";

interface IUser {
  name: string;
}

interface UserModel extends Model<IUser> {
  myStaticMethod(): number;
}

const schema = new Schema<IUser, UserModel>({ name: String });
schema.static("myStaticMethod", function myStaticMethod() {
  return 42;
});

const User = model<IUser, UserModel>("User", schema);

const answer: number = User.myStaticMethod(); // 42
```

# Query Helpers

[查询辅助方法](http://thecodebarbarian.com/mongoose-custom-query-methods.html)让你在 Mongoose 查询上定义自定义的助手函数。
查询辅助方法使用链式语法使查询更具语义。

```javascript
ProjectSchema.query.byName = function (name) {
  return this.find({ name: name });
};
var Project = mongoose.model("Project", ProjectSchema);

// 任何查询，无论是 "find()"、"findOne()"、"findOneAndUpdate()"、"delete()"等，现在都有一个"byName()"助手函数
Project.find().where("stars").gt(1000).byName("mongoose");
```

在 TypeScript 中，Mongoose 的`Model`需要 3 个泛型参数。

1. `DocType`
2. `TQueryHelpers`
3. `TMethods`

第 2 个泛型参数`TQueryHelpers`应该是一个 interface，包含每个查询辅助方法的函数签名。下面是一个创建带有`byName`查询辅助方法的`ProjectModel`的例子:

```typescript
import { Document, Model, Query, Schema, connect, model } from "mongoose";

interface Project {
  name: string;
  stars: number;
}

const schema = new Schema<Project>({
  name: { type: String, required: true },
  stars: { type: Number, required: true },
});

// 查询辅助方法应该返回`Query<any, Document<DocType>> & ProjectQueryHelpers`，以允许链式调用
interface ProjectQueryHelpers {
  byName(name: string): Query<any, Document<Project>> & ProjectQueryHelpers;
}

schema.query.byName = function (
  name
): Query<any, Document<Project>> & ProjectQueryHelpers {
  return this.find({ name: name });
};

// `model()`的第二个参数是要返回的 Model 类
const ProjectModel = model<Project, Model<Project, ProjectQueryHelpers>>(
  "Project",
  schema
);

run().catch((err) => console.log(err));

async function run(): Promise<void> {
  await connect("mongodb://localhost:27017/test");

  // 等价于`ProjectModel.find({ stars: { $gt: 1000 }, name: 'mongoose' })`
  await ProjectModel.find().where("stars").gt(1000).byName("mongoose");
}
```

# Populate

[Mongoose's TypeScript bindings](https://thecodebarbarian.com/working-with-mongoose-in-typescript.html) 在`populate()`中增加了一个泛型参数`Paths`:

```typescript
import { Schema, model, Document, Types } from "mongoose";

interface Parent {
  child?: Types.ObjectId;
  name?: string;
}

const ParentModel = model<Parent>(
  "Parent",
  new Schema({
    child: { type: "ObjectId", ref: "Child" },
    name: String,
  })
);

interface Child {
  name: string;
}

const childSchema: Schema = new Schema({ name: String });
const ChildModel = model<Child>("Child", childSchema);

// 通过 `Paths` 泛型 `{ child: Child }` 重载 `child` 路径来填充
ParentModel.findOne({})
  .populate<{ child: Child }>("child")
  .orFail()
  .then((doc) => {
    const t: string = doc.child.name;
  });
```

另一种方法是定义一个 `PopulatedParent` interface 并使用 `Pick<>` 来提取你要填充的属性。

```typescript
import { Schema, model, Document, Types } from "mongoose";

interface Parent {
  child?: Types.ObjectId;
  name?: string;
}

interface Child {
  name: string;
}

interface PopulatedParent {
  child: Child | null;
}

const ParentModel = model<Parent>(
  "Parent",
  new Schema({
    child: { type: "ObjectId", ref: "Child" },
    name: String,
  })
);

const childSchema: Schema = new Schema({ name: String });
const ChildModel = model<Child>("Child", childSchema);

ParentModel.findOne({})
  .populate<Pick<PopulatedParent, "child">>("child")
  .orFail()
  .then((doc) => {
    const t: string = doc.child.name;
  });
```

# Subdocument

在 TypeScript 中，subdocument 是很棘手的。
默认情况下，Mongoose 将 document interface 中的对象属性视为*嵌套属性*而不是 subdocument。

```ts
import { Schema, Types, model, Model } from "mongoose";

interface Names {
  _id: Types.ObjectId;
  firstName: string;
}

interface User {
  names: Names;
}

type UserModelType = Model<User>;
const userSchema = new Schema<User, UserModelType>({
  names: new Schema<Names>({ firstName: String }),
});
const UserModel = model<User, UserModelType>("User", userSchema);

const doc = new UserModel({ names: { _id: "0".repeat(24), firstName: "foo" } });

// TS: "属性 'ownerDocument' 不存在于 'Names' 类型上"
// 意味着`doc.names`不是一个 subdocument!
doc.names.ownerDocument();
```

Mongoose 提供了一种机制来重载 hydrated document 中的类型。
`Model<>`提供第 3 个泛型参数`TMethodsAndOverrides`: 最初它只是用来定义方法，但你也可以用它来重载类型。
如下所示:

```ts
type UserDocumentOverrides = {
  names: Types.Subdocument<Types.ObjectId> & Names;
};

type UserModelType = Model<User, {}, UserDocumentOverrides>;

const userSchema = new Schema<User, UserModelType>({
  names: new Schema<Names>({ firstName: String }),
});

const UserModel = model<User, UserModelType>("User", userSchema);

const doc = new UserModel({ names: { _id: "0".repeat(24), firstName: "foo" } });
doc.names.ownerDocument(); // 现在 `names` 是一个 subdocument!
```

## subdocument 数组

你也可以使用`TMethodsAndOverrides`来重载数组，以正确定义 subdocument 数组类型。

```ts
interface Names {
  _id: Types.ObjectId;
  firstName: string;
}

interface User {
  names: Names[];
}

type UserDocumentProps = {
  names: Types.DocumentArray<Names>;
};
type UserModelType = Model<User, {}, UserDocumentProps>;

const UserModel = model<User, UserModelType>(
  "User",
  new Schema<User, UserModelType>({
    names: [new Schema<Names>({ firstName: String })],
  })
);

const doc = new UserModel({});
doc.names[0].ownerDocument();
```
