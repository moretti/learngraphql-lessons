```
name:  Defining Queries
bulletPackage: free
```

In previous lessons, we simply invoked queries against a schema that was already defined. We didn't worry about looking at the schema because we wanted to master the query language.

Now, we've a better understanding of the query language. So now is the best time to start writing GraphQL schemas.

Let's get started!

*****

```
id:     setting-up
type:   text
points: 5
```

## Setting up

First we need to download our GraphQL Sandbox and run it locally. That's the place where we will define our schema.

Clone this repo:

~~~
git clone https://github.com/kadirahq/graphql-blog-schema.git
~~~

Then check out the `build-schema` branch:

~~~
cd graphql-blog-schema
git checkout build-schema
~~~

Now install dependencies with:

~~~
npm install
~~~

Start the sandbox with:

~~~
npm start
~~~

Now you can open the sandbox at <http://localhost:3000>.

![](https://cldup.com/MnoG2RvAja.png)

Try to play with it.

*****

```
id:     inspecting-the-schema
type:   mcq
points: 25
```

## Inspecting the schema

Our GraphQL Sandbox has a schema that contains a simple query field called `echo`. It accepts a message as an argument and returns it.

Try running this query:

~~~
{
  receivedMessage: echo(message: "Hello")
}
~~~

You'll get a result like:

~~~
{
  "data": {
    "receivedMessage": "received Hello"
  }
}
~~~

So, let's have a look at the schema. For that, open the following file (inside the cloned repo) in an editor.

* File: `src/schema.js`

It's a self-documented file that uses [ES2015](https://github.com/lukehoban/es6features). We've also used ES2015 [modules](https://github.com/lukehoban/es6features#modules).

At the end of the file, we've defined our schema like this:

~~~
const Schema = new GraphQLSchema({
  query: Query
});
~~~

Here we create a new GraphQL schema object and register the `Query` (we also call this the Root Query). This is what you've seen as `BlogSchema` in the documentation tab of our GraphQL Sandbox. Now let's see what's in this `Query` object.

~~~
const Query = new GraphQLObjectType({
  name: 'BlogSchema',
  description: "Root of the Blog Schema",
  fields: () => ({
    echo: {
      type: GraphQLString,
      description: "Echo what you enter",
      args: {
        message: {type: GraphQLString}
      },
      resolve: function(root, {message}) {
        return `recieved ${message}`;
      }
    }
  })
});
~~~

Here we are creating a new GraphQLObjectType called `BlogSchema`.

We've also given a description for our type. After that, we've some fields in this type. 

We've have one field called `echo`. It's a string (GraphQLString).

> You can look at the top of the `schema.js` to see common types in GraphQL.

This `echo` field also has an argument called `message`.

> It's also possible to add a description to that argument. Here's how to do it:

~~~
{
  ...
  args: {
    message: {type: GraphQLString, description: "Give me a message"}
  }
  ...
}
~~~

Next, we've the resolve function, which is the place where we can implement the logic for this query and return a value.

~~~
{
  ...
  resolve: function(root, args) {
    return `recieved ${args.message}`;
  }
  ...
}
~~~

The second argument of the resolve function comes with values for arguments as defined in the query.

That's all you need to know about our simple GraphQL schema.

---

Now we've a simple task for you. Try to return the following object from the `resolve` function:

~~~
{aa: 10};
~~~

Then invoke a query to get the `echo` field and have a look at the result.

What was the result:

  - {aa: 10}, // Object
  - {"aa": 10} // JSON string
  - **[object Object]**
  - There was an error.

*****

```
id:     answer-inspecting-the-schema
type:   text
points: 5
```

We got the result as **[object Object]**. Here's why?

The `echo` type is a string. Then, GraphQL tries to get a string from the return value of the resolve function. Normally, `[object Object]` is the string value of any object. That's why we get that result.

Try to return other values from different types and inspect the result. Also, try to change the type of the `echo` field and understand how this works.

*****

```
id:     defining-the-post-type
type:   mcq
points: 25
```

## Defining the post type

Now we are going to define an actual type and write a proper root query field. Let's try to implement the `Post` type in our blog and create the root query field `posts`.

Here's the minimal version of the `Post` type in our blog.

~~~
const Post = new GraphQLObjectType({
  name: "Post",
  description: "This represent a Post",
  fields: () => ({
    _id: {type: new GraphQLNonNull(GraphQLString)},
    title: {type: new GraphQLNonNull(GraphQLString)},
    content: {type: GraphQLString}
  })
});
~~~

In this type, we make `_id` and `title` required and make `content` optional. In GraphQL, all the fields we define are optional. If we need to make a field required, we need to mention it explicitly.

Add this type to our `schema.js` file.

> Define it just above our root query type definition.

Now it's time to create the `posts` field in the root query. Here's how to do it:

~~~
const Query = new GraphQLObjectType({
  name: 'BlogSchema',
  description: "Root of the Blog Schema",
  fields: () => ({
    posts: {
      type: new GraphQLList(Post),
      resolve: function() {
        return PostsList;
      }
    }
  })
});
~~~

Here, the `posts` field returns a list of posts (from `Post` type). In the resolve function, we simply return our posts stored in the `PostsList` array. `PostsList` is defined in the `src/data/posts.js` file and you can inspect it.

> Here's how the `schema.js` file looks once you've added all these changes: https://gist.github.com/arunoda/2cbba32de83bfa96099d

Now visit the locally running GraphQL Sandbox and try to query the `posts` field.

> You may need to reload `http://localhost:3000` when you edit the `schema.js` file. Otherwise, autocompletion may not work.

---

Now you've to do a simple task.

Return the following value from the `resolve` function instead of the `PostsList`:

~~~
[{}]
~~~

Now try to invoke the following query:

~~~
{
  posts: posts {
    title
  }
}
~~~

What result did you get?

  - Empty value for the title.
  - **Error as: Cannot return null for non-nullable field Post.title .**
  - Error as: Needs Post.title in the return value.
  - "null" value for the title.

*****

```
id:     answer-defining-the-post-type
type:   text
points: 5
```

## Defining default values

You'll get an error saying, "Cannot return null for non-nullable field Post.title." This is because we defined the `title` field as a required value.

But sometimes in our dataset, we might have some posts with empty values for a title. That's due to some inconstancy in our dataset. But that shouldn't throw an error.

So, in this case we've two options: Either make `title` as an optional value or define a default value for the type. 

For the time being, let's keep the `title` as a required field. Then we can try to add a default value if there is no value for the `title`.

~~~
const Post = new GraphQLObjectType({
  name: "Post",
  description: "This represent a Post",
  fields: () => ({
    _id: {type: new GraphQLNonNull(GraphQLString)},
    title: {
      type: new GraphQLNonNull(GraphQLString),
      resolve: function(post) {
        return post.title || "Does not exists";
      }
    },
    content: {type: GraphQLString}
  })
});
~~~

Have a look at the `resolve` function of the `title` field. Here we specifically defined a resolve function to provide the content for the title field.

First argument of the resolve function is an object. It's one of the values in the `PostsList` array.

Then we can customize the actual value in the data source and return anything we need.

*****
```
id:  defining-nested-fields
type: mcq
points: 20
```

## Defining nested fields

Now we are trying to define a nested field into our schema. For an example, let's try to add an `author` field to our `Post` type.

For that, first we need to create the `Author` type.

~~~
const Author = new GraphQLObjectType({
  name: "Author",
  description: "This represent an author",
  fields: () => ({
    _id: {type: new GraphQLNonNull(GraphQLString)},
    name: {type: GraphQLString}
  })
});
~~~

Then let's define the `author` field in our `Post` type.

~~~
const Post = new GraphQLObjectType({
  name: "Post",
  description: "This represent a Post",
  fields: () => ({
    ...
    author: {
      type: Author,
      resolve: function(post) {
        return _.find(AuthorsList, a => a._id == post.author);
      }
    }
  })
});
~~~

In our `PostsList` dataset, each object(post) has a string value for the "author". That's the `_id` of the author, which is stored in `AuthorsList`. 

So, the resolve function of the `author` field simply finds the correct author and returns him. We've also defined the type of the `author` field as `Author`.

> Here's what the final `schema.js` file looks like: <https://gist.github.com/arunoda/c29128be2c5e979475ec>.

---

Now invoke the following query:

~~~
{
  posts {
    title,
    author {
      name
    }
  }
}
~~~

How many posts were created by author "Kasun Indi":

  - **1**
  - 2
  - 3
  - 4

*****
```
id:  finally
type: text
points: 5
```

## Finally

Now you know how to define a GraphQL schema and resolve fields. We've also discussed how to write nested fields. In this way you can define as many nested fields as you like.

That's how you build your dataset as a graph. We'll explore more about GraphQL schemas in upcoming lessons.
