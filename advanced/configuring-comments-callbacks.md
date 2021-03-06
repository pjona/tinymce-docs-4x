---
layout: default
title: Configuring callbacks for Comments 2.0
title_nav: Configuring callbacks for Comments 2.0
description: Instructions for configuring callbacks for Comments 2.0
keywords: comments commenting tinycomments callback
---

## Introduction

**Callback mode** is the default mode for [Comments 2.0]({{site.baseurl}}/plugins/comments/comments_2.0/). In the callback mode, the user needs to configure storage to be able to save comments on the server. The Comments functions (create, reply, edit, delete comment, delete all conversations, and lookup) are configured differently depending upon the server-side storage configuration.

### Required settings

Comments 2.0 requires the following functions to be defined:

```js
tinymce.init({
  ...
   tinycomments_create: function (req, done, fail) { ... },
   tinycomments_reply: function (req, done, fail) { ... },
   tinycomments_delete: function (req, done, fail) { ... },
   tinycomments_delete_all: function (req, done, fail) { ... },
   tinycomments_delete_comment: function (req, done, fail) { ... },
   tinycomments_lookup: function (req, done, fail) { ... },
   tinycomments_edit_comment: function (req, done, fail) { ... },
});

```

All functions incorporate `done` and `fail` callbacks as parameters. The function return type is not important, but all functions must call exactly one of these two callbacks: `fail` or `done`.

* The `fail` callback takes either a string or a JavaScript Error type.

* The `done` callback takes an argument specific to each function.

Most (create, reply, and edit) functions require configuring the **current author**:

* **Current author** - Comments 2.0 does not know the name of the current user. After a user comments (triggering `tinycomments_create` for the first comment, or `tinycomments_reply` for subsequent comments), Comments 2.0 requests the updated conversation via `tinycomments_lookup`, which should now contain the additional comment with the proper author. Determining the current user and storing the comment related to that user, has to be configured by the user.

### tinycomments_create

Comments 2.0 uses the conversation `tinycomments_create` function to create a comment.

The `tinycomments_create` function saves the comment as a new conversation and returns a unique conversation ID via the `done` callback. If an unrecoverable error occurs, it should indicate this with the fail callback.

The `tinycomments_create` function is given a request (req) object as the first parameter, which has these fields:

* **content**: the content of the comment to create.

* **createdAt**: the date the comment was created.

The done callback needs to take an object of the form:

```js
{
  conversationUid: the new conversation uid 
}
```

### tinycomments_reply

Comments 2.0 uses the conversation `tinycomments_reply` function to reply to a comment.

The `tinycomments_reply` function saves the comment as a reply to an existing conversation and returns via the `done` callback once successful. Unrecoverable errors are communicated to TinyMCE by calling the `fail` callback instead.

The `tinycomments_reply` function is given a request (req) object as the first parameter, which has these fields:

* **conversationUid**: the uid of the conversation the reply is a part of.

* **content**: the content of the comment to create.

* **createdAt**: the date the comment was created.

The done callback needs to take an object of the form:

```js
{
  commentUid: the value of the new comment uid
}
```

### tinycomments_edit_comment

Comments 2.0 uses the conversation `tinycomments_edit_comment` function to edit a comment.

The `tinycomments_edit_comment` function allows updating or changing original comments and returns via the `done` callback once successful. Unrecoverable errors are communicated to TinyMCE by calling the `fail` callback instead.

The `tinycomments_edit_comment` function is given a request (req) object as the first parameter, which has these fields:

* **conversationUid**: the uid of the conversation the reply is a part of.

* **commentUid**: the uid of the comment to edit (it can be the same as `conversationUid` if editing the first comment in a conversation)

* **content**: the content of the comment to create.

* **modifiedAt**: the date the comment was modified.

The done callback needs to take an object of the form:

```js
{
  canEdit: whether or not the Edit succeeded
  reason: an optional string explaining why the edit was not allowed (if canEdit is false)
}
```

### tinycomments_delete

The `tinycomments_delete` function should asynchronously return a flag indicating whether the comment/comment thread was removed using the `done` callback. Unrecoverable errors are communicated to TinyMCE by calling the `fail` callback instead.

The `tinycomments_delete` function is given a request (req) object as the first parameter, which has this field:

* **conversationUid**: the uid of the conversation the reply is a part of.

The done callback needs to take an object of the form:

```js
{
  canDelete:boolean
  reason: an optional string explaining why the delete was not allowed (if canDelete is false)
}
```

> Note: Failure to delete due to permissions or business rules is indicated by "false", while unexpected errors should be indicated using the "fail" callback.

### tinycomments_delete_all

The `tinycomments_delete_all` function should asynchronously return a flag indicating whether all the comments in a conversation were removed using the `done` callback. Unrecoverable errors are communicated to TinyMCE by calling the `fail` callback instead.

The `tinycomments_delete_all` function is given a request (req) object as the first parameter with no fields.

The done callback needs to take an object of the form:

```js
{
  canDelete:boolean
  reason: an optional string explaining why the deleteAll was not allowed (if canDelete is false)
}
```

> Note: Failure to delete due to permissions or business rules is indicated by "false", while unexpected errors should be indicated using the "fail" callback.

### tinycomments_delete_comment

The `tinycomments_delete_comment` function should asynchronously return a flag indicating whether the comment/comment thread was removed using the `done` callback. Unrecoverable errors are communicated to TinyMCE by calling the `fail` callback instead.

The `tinycomments_delete_comment` function is given a request (req) object as the first parameter, which has these fields:

* **conversationUid**: the uid of the conversation the reply is a part of.
* **commentUid**: the uid of the comment to delete (cannot be the same as conversationUid)

The done callback needs to take an object of the form:

```js
{
  canDelete:boolean
  reason: an optional reason
}
```

> Note: Failure to delete due to permissions or business rules is indicated by "false", while unexpected errors should be indicated using the "fail" callback.


### tinycomments_lookup
Comments 2.0 uses the Conversation `tinycomments_lookup` function to retrieve an existing conversation via a conversation unique ID.

The **Display names** configuration must be considered for the `tinycomments_lookup` function:

* **Display names** - Comments 2.0 uses a simple string for the display name. For the `lookup` function, Comments 2.0 expects each comment to contain the author's display name, not a user ID, as Comments 2.0 does not know the user identities. The `lookup` function should be implemented considering this and resolve user identifiers to an appropriate display name.

The conventional conversation object structure that should be returned via the `done` callback is as follows:

The `tinycomments_lookup` function is given a request (req) object as the first parameter, which has this field:

* **conversationUid**: the uid of the conversation the reply is a part of.

The done callback needs to take an object of the form:

```js
{
 comments: [
   {
     author: 'Demouser1',
     createdAt: 'date',
     content: 'Starter',
     modifiedAt: 'date',
     uid: 'asfasdf87dfas08asd0fsadflsadf'
   },
   {
    author: 'Demouser2',
    createdAt: 'date',
    content: 'Reply',
    modifiedAt: 'date',
    uid: 'asfasdf87dfas08asd0fsadflsadg’'
   },
 ]
}

 ]
}

```

For more information on Comments commercial feature, visit our [Premium Features]({{ site.baseurl }}/enterprise/tiny-comments/) page.
