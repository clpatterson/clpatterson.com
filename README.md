### Installing on a new machine
```zsh
brew install hugo
```
Then setup the theme
```zsh
cd themes/
git clone https://github.com/clpatterson/ezhil-fork.git
```

### Running
```zsh
hugo server
```
This will start the development server at http://localhost:1313 with live reload enabled.

### Developement
- changes to `config.toml` require a full reload because configs are loaded on start.
- I find the hugo docs confusing sometimes. The [Giraffe Academy](https://www.giraffeacademy.com/static-site-generators/hugo/installing-using-themes/) docs are a helpful alternative.

### Deploy
- Just commit the new compiled `public/` directory after making a change.
- Check [github actions logs](https://github.com/clpatterson/clpatterson.com/actions) to see build succeeded. 

