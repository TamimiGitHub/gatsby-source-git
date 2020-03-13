# gatsby-source-git

Source plugin for pulling files into the Gatsby graph from abitrary Git repositories (hosted anywhere). This is useful if the markdown files you wish to render can't live within your gatsby codebase, or if need to aggregate content from disparate repositories.

It clones the repo(s) you configure (a shallow clone, into your cache folder if
you're interested), and then sucks the files into the graph as `File` nodes, as
if you'd configured
[`gatsby-source-filesystem`](https://www.gatsbyjs.org/packages/gatsby-source-filesystem/)
on that directory. As such, all the tranformer plugins that operate on files
should work exactly as they do with `gatsby-source-filesystem` eg with
`gatsby-transformer-remark`, `gatsby-transformer-json` etc.

The only difference is that the `File` nodes created by this plugin will
also have a `gitRemote` field, which will provide you with various bits of
Git related information. The fields on the `gitRemote` node are
mostly provided by
[IonicaBazau/git-url-parse](https://github.com/IonicaBizau/git-url-parse), with
the addition of `ref` and `weblink` fields, which are
the 2 main things you probably want if you're constructing "edit on github"
style links.

N.B. Although with respect to sourcing this works as a drop-in replacement for `gatsby-source-filesystem`, there are a number of helpers included in that module (`createFilePath`, `createRemoteFileNode`, `createFileNodeFromBuffer`) that are not duplicated here – but you can still import and use them from there as needed.

## Requirements

Requires [git](http://git-scm.com/downloads) to be installed, and to be callable using the command `git`.

Ideally we'd use [nodegit](https://github.com/nodegit/nodegit), but it doesn't support shallow clones (see [libgit2/libgit2#3058](https://github.com/libgit2/libgit2/issues/3058)) which would have a significant effect on build times if you wanted to read files from git repositories with large histories.

Only public repositories are supported right now. But a PR should be simple enough if you want that.

## Install

`npm install --save gatsby-source-git`

## How to use

```javascript
// In your gatsby-config.js
module.exports = {
  plugins: [
    // You can have multiple instances of this plugin
    // to read source nodes from different repositories.
    {
      resolve: `gatsby-source-git`,
      options: {
        name: `repo-one`,
        remote: `https://bitbucket.org/stevetweeddale/markdown-test.git`,
        // Optionally supply a branch. If none supplied, you'll get the default branch.
        branch: `develop`,

        // (Optional) Specify a local path for the repo. If omitted, it will
        // default to using a directory within the local Gatsby cache.
        // local: '/explicit/path/to/repo-one',

        // Tailor which files get imported eg. import the docs folder from a codebase.
        patterns: `docs/**`
      }
    },
    {
      resolve: `gatsby-source-git`,
      options: {
        name: `repo-two`,
        remote: `https://bitbucket.org/stevetweeddale/markdown-test.git`,
        // Multiple patterns and negation supported. See https://github.com/mrmlnc/fast-glob
        patterns: [`*`, `!*.md`]
      }
    }
  ]
};
```

This will result in `File` nodes being put in your data graph, it's then up to you to do whatever it is you want to do with that data.

## How to query

You can query file nodes exactly as you would node query for nodes created with
[`gatsby-source-filesystem`](https://www.gatsbyjs.org/packages/gatsby-source-filesystem/),
eg:

```graphql
{
  allFile {
    edges {
      node {
        extension
        dir
        modifiedTime
      }
    }
  }
}
```

Similarly, you can filter by the `name` you specified in the config by using
`sourceInstanceName`:

```graphql
{
  allFile(filter: { sourceInstanceName: { eq: "repo-one" } }) {
    edges {
      node {
        extension
        dir
        modifiedTime
      }
    }
  }
}
```

And access some information about the git repo:

```graphql
{
  allFile {
    edges {
      node {
        gitRemote {
          webLink
          ref
        }
      }
    }
  }
}
```

## Creating pages

If you want to programatically create pages on your site from the files in your git repo, you should be able to follow the standard examples, such as [part 7 of the Gatsby tutorial](https://www.gatsbyjs.org/tutorial/part-seven/) or [the standard docs page](https://www.gatsbyjs.org/docs/creating-and-modifying-pages/#creating-pages-in-gatsby-nodejs).
