# （redux-orm）官方文档

## Installation

安装该包有两种方式：npm方式、浏览器方式：

npm方式：

```
npm install redux-orm --save
```

浏览器方式：

```html
<script src="https://tommikaikkonen.github.io/redux-orm/dist/redux-orm.js"></script>
```

版本下载：

* 完整版：[https://tommikaikkonen.github.io/redux-orm/dist/redux-orm.js](https://tommikaikkonen.github.io/redux-orm/dist/redux-orm.js)
* 最小版：[https://tommikaikkonen.github.io/redux-orm/dist/redux-orm.min.js](https://tommikaikkonen.github.io/redux-orm/dist/redux-orm.min.js)

## 1.Usage

### 1.1.Declare Your Models

您可使用ES6的语法定义模型【Model】，直接从`Model`继承过来，您需要定义模型的所有关联字段，所有数据字段推荐定义成带`Getter/Setter`的字段，它们可以直接在初始化Model时设置。`redux-orm`支持一对一、多对多的关系，它可使用外键（`oneToOne`，`many`和`fk`导入），普通属性访问方式和JavaScript对象一样。

```javascript
// models.js
import {fk, many, attr, Model} from 'redux-orm';

class Book extends Model {
    toString() {
        return `Book: ${this.name}`;
    }
    // Declare any static or instance methods you need.
}
Book.modelName = 'Book';

// Declare your related fields.
Book.fields = {
    id: attr(), // non-relational field for any value; optional but highly recommended
    name: attr(),
    authors: many('Author', 'books'),
    publisher: fk('Publisher', 'books'),
};
```

### 1.2. Register Models and Generate an Empty Database State

在模型中定义字段指定了数据库中的表结构，为了生成数据库结构的描述，您需要统一的地方来注册这些Model。一个ORM类的实例可以注册所有的Model以及生成针对该Model的Schema信息并且将这些信息映射到数据库中。通常您需要一个文件来导入一个单独的ORM实例（创建统一创建中心，单件模式）：

```javascript
// orm.js
import { ORM } from 'redux-orm';
import { Book, Author, Publisher } from './models';

const orm = new ORM();
orm.register(Book, Author, Publisher);

export default orm;
```

您同样可以在同一个文件中注册ORM实例的模型，并且全部Export出来。现在您注册了一个Model，接下来可以生成空数据库状态，当前这个对象是一个普通对象，嵌入的是一个JavaScript：

```javascript
// index.js

import orm from './orm';

const emptyDBState = orm.getEmptyState();
```

### 1.3. Applying Updates to the Database

当您有了一个数据库的状态，则您可以开启一个ORM会话并修改其值。ORM实例提供了一个`session`方法：传入一个数据库状态，返回一个Session实例。

```javascript
const session = orm.session(emptyDBState);
```

使用Session的已注册模型本身可以用session对象的属性访问：

```javascript
const Book = session.Book;
```

Model提供了一个接口用来查询/更新数据库状态：

```javascript
Book.withId(1).update({ name: 'Clean Code' });
Book.all().filter(book => book.name === 'Clean Code').delete();
Book.hasId(1)
// false
```

初始化数据状态是不会突变的，一个新的数据库状态以及更新可以反映在Session实例中：

```javascript
const updatedDBState = session.state;
```

## 2. Integration

### 2.1. Redux Integration

Redux-ORM和Redux的集成是很基础的，您只需要定义一个reducer用来实例化Redux中的状态对应的数据库会话即可，然后当您需要更新内容时，您可以直接返回下一个状态的会话。

```javascript
import { orm } from './models';
function ormReducer(dbState, action) {
    const sess = orm.session(dbState);

    // Session-specific Models are available
    // as properties on the Session instance.
    const { Book } = sess;

    switch (action.type) {
    case 'CREATE_BOOK':
        Book.create(action.payload);
        break;
    case 'UPDATE_BOOK':
        Book.withId(action.payload.id).update(action.payload);
        break;
    case 'REMOVE_BOOK':
        Book.withId(action.payload.id).delete();
        break;
    case 'ADD_AUTHOR_TO_BOOK':
        Book.withId(action.payload.bookId).authors.add(action.payload.author);
        break;
    case 'REMOVE_AUTHOR_FROM_BOOK':
        Book.withId(action.payload.bookId).authors.remove(action.payload.authorId);
        break;
    case 'ASSIGN_PUBLISHER':
        Book.withId(action.payload.bookId).publisher = action.payload.publisherId;
        break;
    }

    // the state property of Session always points to the current database.
    // Updates don't mutate the original state, so this reference is not
    // equal to `dbState` that was an argument to this reducer.
    return sess.state;
}
```

原来的Redux-ORM主张通过在Model类上附加一个静态reducer函数来实现特定Model的reducer，若要在Model类中定义更新逻辑，可以在模型上指定reducer静态方法，该方法作为第一个参数接收、特定模型会话作第二个参数、整个会话作为第三个参数。

```javascript
class Book extends Model {
    static reducer(action, Book, session) {
        switch (action.type) {
        case 'CREATE_BOOK':
            Book.create(action.payload);
            break;
        case 'UPDATE_BOOK':
            Book.withId(action.payload.id).update(action.payload);
            break;
        case 'REMOVE_BOOK':
            const book = Book.withId(action.payload);
            book.delete();
            break;
        case 'ADD_AUTHOR_TO_BOOK':
            Book.withId(action.payload.bookId).authors.add(action.payload.author);
            break;
        case 'REMOVE_AUTHOR_FROM_BOOK':
            Book.withId(action.payload.bookId).authors.remove(action.payload.authorId);
            break;
        case 'ASSIGN_PUBLISHER':
            Book.withId(action.payload.bookId).publisher = action.payload.publisherId;
            break;
        }
        // Return value is ignored.
        return undefined;
    }

    toString() {
        return `Book: ${this.name}`;
    }
}
```

为Redux获取reducer则可调用这些`reducer`方法：

```javascript
import { createReducer } from 'redux-orm';
import { orm } from './models';

const reducer = createReducer(orm);
```

`createReducer`很简单，这里可以看看它的源代码：

```javascript
function createReducer(orm, updater = defaultUpdater) {
    return (state, action) => {
        const session = orm.session(state || orm.getEmptyState());
        updater(session, action);
        return session.state;
    };
}

function defaultUpdater(session, action) {
    session.sessionBoundModels.forEach(modelClass => {
        if (typeof modelClass.reducer === 'function') {
            modelClass.reducer(action, modelClass, session);
        }
    });
}
```

如您所见，它只是实例化一个新的Session，循环遍历会话中的所有Model，并调用reducer方法（如果存在），然后它返回应用了所有更新的新数据库状态。

### 2.2. Use with React

您可以使用内存选择器来查询状态，`redux-orm`使用了智能记忆功能：下边选择器访问了Author、AuthorBook分支（AuthorBook是从模型字段声明生成的多对多分支），只有当这些分支更改时，才会重新计算选择器。访问的分支在第一次运行时会被解析。

```javascript
// selectors.js
import schema from './schema';
const authorSelector = schema.createSelector(session => {
    return session.Author.map(author => {

        // Returns a reference to the raw object in the store,
        // so it doesn't include any reverse or m2m fields.
        const obj = author.ref;
        // Object.keys(obj) === ['id', 'name']

        return Object.assign({}, obj, {
            books: author.books.withRefs.map(book => book.name),
        });
    });
});

// Will result in something like this when run:
// [
//   {
//     id: 0,
//     name: 'Tommi Kaikkonen',
//     books: ['Introduction to redux-orm', 'Developing Redux applications'],
//   },
//   {
//     id: 1,
//     name: 'John Doe',
//     books: ['John Doe: an Autobiography']
//   }
// ]
```

使用`createSelector`创建的选择器可以用来作为使用了reselect的其他任何选择器的输入，它们也很适合使用`redux-thunk`：调用`getState()`获取整个状态，将ORM分支传递给选择器，并读取结果。一个很好的用例是将数据序列化为第三方API调用的自定义格式。

因为选择器内存化了（被记忆了），您可以在React使用纯渲染来获得性能。

```javascript
// components.js
import PureComponent from 'react-pure-render/component';
import { authorSelector } from './selectors';
import { connect } from 'react-redux';

class App extends PureComponent {
    render() {
        const authors = this.props.authors.map(author => {
            return (
                <li key={author.id}>
                    {author.name} has written {author.books.join(', ')}
                </li>
            );
        });

        return (
            <ul>
                {authors}
            </ul>
        );
    }
}

function mapStateToProps(state) {
    return {
        authors: authorSelector(state.orm),
    };
}

export default connect(mapStateToProps)(App);
```

## 3.理解redux-orm

### ORM？

`redux-orm`可以处理关联数据，将数据结构化成ER数据库，数据库在这种情况实际上是一个JavaScript对象。

### Why？

对于简单应用，通常会手写reducers，但是如果您定义的对象类型不断增加，则您需要维护它们之间的关系。ImmutableJS已经很大程度降低了复杂度，但`redux-orm`主要用来指定关系数据库。

### Immutability

如果我们从一个初始化数据状态的会话开始，则需要更新对应名称。

首先，开启一个新会话：

```javascript
import { orm } from './models';

const dbState = getState().db; // getState() returns the redux state.
const sess = orm.session(dbState);
```

该会话维护了数据库状态的一个引用，我们没有更新数据库状态，因此它们依然是相等的：

```js
sess.state == dbState
// true
```

然后可执行更新：

```javascript
const book = sess.Book.withId(1)

book.name // 'Refactoring'
book.name = 'Clean Code'
book.name // 'Clean Code'

sess.state === dbState
// false.
```

更新执行后，因为会话并不会修改原始状态，他是直接封装`sess.state`然后创建一个新状态，而更新原始状态可使用：

```javascript
// Save this reference so we can compare.
const updatedState = sess.state;

book.name = 'Patterns of Enterprise Application Architecture'

sess.state === updatedState
// true. If possible, future updates are applied with mutations. If you want
// to avoid making mutations to a session state, take the session state
// and start a new session with that state.
```

若有可能，会执行相关更新。这种情况，数据库已经被改过了，所以并不需要修改引用。若您想避免修改会话状态，则可以基于当前状态开启一个新状态。

### Customizability

和您继承`Model`一样，您可以针对`QuerySet`做同样的事（自定义Model实例的集合），您也可以自己实现整个数据库。

### Caveats

ORM抽象和手工编写的reducer相比，永远不会那么有效，并且增加了项目的构建总大小（最后检查过，最小化源文件，转换成gzip文件大小约8KB），若果您有非常简单的不包含关系的数据，则`redux-orm`就过度了，所以它大大提高了开发遍历。

### 4.1. ORM

您可以查阅所有的ORM文档：[Document](http://tommikaikkonen.github.io/redux-orm/ORM.html)

初始化

```javascript
const orm = new ORM();
```

实例化方法：

* `register(...models: Array<Model>)`注册Model类到ORM实例；
* `session(state: any)`开启一个带状态`state`的会话`Session`。

### 4.2. Redux Integration

* `createReducer(orm: ORM)`：返回一个可以插入Redux的reducer函数，这个reducer将会返回数据库的下一个状态，调用它之前您只需要注册您的Model类。
* `createSelector(orm: ORM, [...inputSelectors], selectorFunc)`：返回一个可记忆的selector函数`selectorFunc`，它使用`session`作第一参数，接着使用任意`inputSelectors`作为第二参数。

### 4.3. Model

您可以查看Model详细文档。

**Instantiation**：不要直接创建Model，尽可能使用`create`创建。

**Class Methods：**

* `hasId(id)`：检查数据库中是否包含了当前id对应的状态记录。
* `withId(id)`：根据id读取Model的实例。
* `get(matchObj)`：根据匹配属性`matchObj`读取相匹配的Model实例。
* `create(props)`：可使用`props`创建一个新的Model，如果没有提供id，则新的id应该是`Math.max(...allOtherIds) + 1`。

**Instance Attributes**：

* `ref`：返回JavaScript对象的直接引用，用于描述Model实例。

**Instance Methods**：

* `equals(otherModel)`：判断当前Model和`otherModel`是否相等，相等表示所有的属性值都相等。
* `set(propertyName,value)`：更新`propertyName`属性的值为`value`，返回值是`Undefined`，等价于通常赋值语句。
* `update(mergeObj)`：将`mergeObj`中的数据合并到当前Model中，返回`Undefined`（该方法存在引用修改）。
* `delete()`：从数据库中删除当前Model实例记录，返回`Undefined`。

**Subclassing**

您可以直接使用ES6中的语法从`Model`实现继承，任何定义的实例方法在Model实例中都是有效的，而任何静态方法则是在Class类级别有效。针对关联字段定义，可设置类中的`fields`来实现，也可以像下边这种方式来实现：

```javascript
class Book extends Model {
    static get fields() {
        return {
            id: attr(),
            name: attr(),
            author: fk('Author'),
        };
    }
}
// alternative:
Book.fields = {
    id: attr(),
    name: attr(),
    author: fk('Author'),
}
```

所有的fk、oneToOne、many都接收一个参数（它关联的Model名），这些字段将作为每个Model实例上的属性提供，您可以使用相关实例的id值或相关实例本身设置该关联实例的字段。

对于fk，您可以通过`author.bookSet`访问相反的关系，其中相关名称为`${modelName}Set`，对许多人也是如此，对于`oneToOne`，只能通过字段上声明的模型名称来访问反向关系：`author.book`。

对于many，访问Model实例上的字段将会返回一个`QuerySet`，另外还有两个方法`add`和`remove`，它们获取一个或多个参数，其中参数是模型实例或其ID，调用这些方法将会返回存储中的记录的下一个状态。

声明模型类时，始终记住设置`modelName`，它必须被明确设置，并且勇于解析所有相关字段。

**Declaring **`modelName`：

```javascript
class Book extends Model {
    static get modelName() {
        return 'Book';
    }
}
// alternative:
Book.modelName = 'Book';
```

**Declaring **`options`：

若您想要针对`redux-orm`数据库指定选项，则您可以在Model类中定义一个静态`options`属性，通常使用id定义属性名表示键值。

```javascript
// This is the default value. 
Book.options = {
    idAttribute: 'id',
};
```

### 4.4. QuerySet

您可以调用所有`Model`类中拥有的功能，因为它们都是`Model`的类方法，这种情况下`QuerySet`中一般是所有的`Model`实例。

**Instance Methods**：

* `toRefArray()`：返回`QuerySet`中用来描述的一个JavaScript对象的数组，这些对象会直接引用存储中的数据。
* `toModelArray()`：返回`QuerySet`中一个`Model`实例对象的数组。
* `count()`：返回`QuerySet`中`Model`实例的数量。
* `exists()`：如果包含实体数量大于0则返回true，反之返回false。
* `filter(filterArg)`：根据传入的filter从原始`QuerySet`中生成一个新的`QuerySet`。对于filterArg参数，您可以传入一个`redux-orm`相匹配的实体，或者传入一个自定义函数，返回true则将该实体放到新的`QuerySet`中，反之则排除。
* `exclude`：【略】，和`filter(filterArg)`功能相反。
* `all()`：返回一个`QuerySet`的新拷贝。
* `at(index)`：根据索引读取`QuerySet`对应位置上的`Model`实例。
* `first()`：读取第一个`Model`，index = 0。
* `last()`：读取最后一个`Model`，index = `querySet.count() - 1`。
* `delete()`：删除所有实体。
* `update(mergedObj)`：更新所有`QuerySet`中的实体，将传入的对象和这些实体一一合并。

### 4.5. Session

**Instantiation**：您不需要自己创建该对象，直接使用`orm.session`即可。

**Instance Properties**：state：当前数据库对应数据在会话中的状态。

另外，您可以访问所有Schema中注册过的Model，查询以及更新对应实例属性，如：

```javascript
const session = orm.session(state);
session.Book // Model class: Book
session.Author // Model class: Author
session.Book.create({id: 5, name: 'Refactoring', release_year: 1999});
```



