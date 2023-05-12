# heehong.github.io


- template 적용방법 
- 순서대로 진행
brew update
brew install rbenv ruby-build

rbenv versions
rbenv install -l

- install 에러시 진행
brew install libyaml

rbenv install 3.2.2 

rbenv local 3.2.2
rbenv global 3.2.2

- 권한 에러시 
- ERROR:  While executing gem ... (Gem::FilePermissionError)
- You don't have write permissions for the /Library/Ruby/Gems/2.6.0 directory.
source ~/.bash_profile
export GEM_HOME="$HOME/.gem"

gem install jekyll bundler
gem install jekyll

- github.io 위치에서 실행
- 에러시 Gemfile.lock 제거 후 다시 실행
bundle install
bundle exec jekyll serve