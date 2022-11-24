# Tutorial
GitHub has a JSON API, so let's play with that. This URL gets us the last 5 commits from the jq repo.

```shell
curl 'https://api.github.com/repos/stedolan/jq/commits?per_page=5'
```

GitHub returns nicely formatted JSON. For servers that don't, it can be helpful to pipe the response through jq to pretty-print it. The simplest jq program is the expression ., which takes the input and produces it unchanged as output.

```shell
curl 'https://api.github.com/repos/stedolan/jq/commits?per_page=5' | jq '.'
```

We can use jq to extract just the first commit.

```shell
curl 'https://api.github.com/repos/stedolan/jq/commits?per_page=5' | jq '.[0]'
```

For the rest of the examples, I'll leave out the curl command - it's not going to change.

There's a lot of info we don't care about there, so we'll restrict it down to the most interesting fields.

```shell
jq '.[0] | {message: .commit.message, name: .commit.committer.name}'
```

The | operator in jq feeds the output of one filter (.[0] which gets the first element of the array in the response) into the input of another ({...} which builds an object out of those fields). You can access nested attributes, such as .commit.message.

Now let's get the rest of the commits.

```shell
jq '.[] | {message: .commit.message, name: .commit.committer.name}'
```

.[] returns each element of the array returned in the response, one at a time, which are all fed into {message: .commit.message, name: .commit.committer.name}.

Data in jq is represented as streams of JSON values - every jq expression runs for each value in its input stream, and can produce any number of values to its output stream.

Streams are serialised by just separating JSON values with whitespace. This is a cat-friendly format - you can just join two JSON streams together and get a valid JSON stream.

If you want to get the output as a single array, you can tell jq to "collect" all of the answers by wrapping the filter in square brackets:

```shell
jq '[.[] | {message: .commit.message, name: .commit.committer.name}]'
```

Next, let's try getting the URLs of the parent commits out of the API results as well. In each commit, the GitHub API includes information about "parent" commits. There can be one or many.

```json
"parents": [
  {
    "sha": "54b9c9bdb225af5d886466d72f47eafc51acb4f7",
    "url": "https://api.github.com/repos/stedolan/jq/commits/54b9c9bdb225af5d886466d72f47eafc51acb4f7",
    "html_url": "https://github.com/stedolan/jq/commit/54b9c9bdb225af5d886466d72f47eafc51acb4f7"
  },
  {
    "sha": "8b1b503609c161fea4b003a7179b3fbb2dd4345a",
    "url": "https://api.github.com/repos/stedolan/jq/commits/8b1b503609c161fea4b003a7179b3fbb2dd4345a",
    "html_url": "https://github.com/stedolan/jq/commit/8b1b503609c161fea4b003a7179b3fbb2dd4345a"
  }
]
```

We want to pull out all of the "html_url" fields inside that array of parent commits and make a simple list of strings to go along with the "message" and "author" fields we already have.

```shell
jq '[.[] | {message: .commit.message, name: .commit.committer.name, parents: [.parents[].html_url]}]'
```

Here we're making an object as before, but this time the parents field is being set to [.parents[].html_url], which collects all of the parent commit URLs defined in the parents object.
