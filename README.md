# leanpub-orb

CircleCI orb for automating Leanpub book preview/publishing

## Leanpub

[Leanpub](https://leanpub.com/) is a self-publishing service that supports ebooks and online courses. These can be written in Markdown or Markua and their builds and publishing automated using the [bLeanpub API](https://leanpub.com/help/api).   After you register for a key, you can use the API with a webhook to trigger a build directly.

## Workflow

My preferred workflow is outlined below; this can be directly triggered from git, saving time and effort:

- a *subset preview* is triggered for every commit to your book's repository;
- a *regular preview* is triggered for commits with tags beginning with `preview`;
- a *silent publish* is triggered for commits with tags beginning with `silent-publish`. This publishes the book but does not send out release notes;
- a *publish* is triggered for commits with tags beginning with `publish`. This publishes the book and sends out release notes to readers who have opted to receive them. These notes are taken either from description of the tag, if the tag is annotated, or from the body of the commit message, if the tag is a regular tag.

## Usage

To use, add a `.circleci/config.yml` file to your repo, containing the following code:

```
version: 2.1

orbs:
  leanpub: zzamboni/leanpub@0.2.4

# This tag-based book building workflow dispatches to the correct job
# depending on tagging
workflows:
  version: 2
  build-book:
    jobs:
      - leanpub/subset-preview:
          filters:
            tags:
              ignore:
                - /^preview.*/
                - /^publish.*/
                - /^silent-publish.*/
      - leanpub/full-preview:
          filters:
            tags:
              only: /^preview.*/
            branches:
              ignore: /.*/
      - leanpub/auto-publish:
          name: leanpub/silent-publish
          auto-release-notes: false
          filters:
            tags:
              only: /^silent-publish.*/
            branches:
              ignore: /.*/
      - leanpub/auto-publish:
          auto-release-notes: true
          filters:
            tags:
              only: /^publish.*/
            branches:
              ignore: /.*/
```

Ensure your repo name corresponds to your Leanpub book slug. If not, add a `book-slug` parameter to each job definition.

Next, enable CircleCI by:

- disabling your current webhooks, if any
- login to (CircleCI)[https://circleci.com]
- set up your repo through the 'Add Project' screen. Skip the `config.yml` and define `LEANPUB_API_KEY` as an environment variable in your CircleCI project.
- on the 'Workflows' screen, you can rerun your workflow, or make a commit on your repo to trigger the CircleCI build.

You can find some additional descriptions and tips here: https://zzamboni.org/post/automating-leanpub-book-publishing-with-hammerspoon-and-circleci/
