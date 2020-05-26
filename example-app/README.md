# Example Gitbook App

This is a gitbook project. Check out the [official documentation](https://toolchain.gitbook.com/) for more detatis.

The content of the book is in the `./content` folder in this project.

### To edit and add in the project

- SUMMARY.md is the left-side menu. Add the link to new pages to add to the menu.
- Create and edit in the `./content` folder. This will automatically add it to the `_book` folder.

To install the dependencies:

```bash
npm install
```

Run the gitbook server at `localhost:4000`

```bash
npm run serve
```

Build a static copy of the book:

```bash
npm run build
```

Run a local debug version of the server:

```bash
npm run debug
```
