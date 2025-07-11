## Documentation
This repository hosts the [Documentation](https://netick.net/docs/2/index.html) for Netick.

## Contributing  Guide
The documentation of Netick is built using [docfx](https://github.com/dotnet/docfx). The documentation articles are located at `articles`, and they are written using markdown format.


### Getting Started

Installing docfx:
```
dotnet tool install -g docfx
```

Serving the docs on localhost:
```
docfx --serve
```


### Adding New Articles

Make a new page at `articles`, and link it in `articles\toc.md`.