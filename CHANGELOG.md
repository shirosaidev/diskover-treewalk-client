# Diskover Treewalk Client Change Log

## [1.0.23] = 2020-04-10
### added
- -E --noexcludeddirs cli option to not exclude any directories sent to diskover proxy

## [1.0.22] = 2019-01-19
### fixed
- bug with directory excludes and pscandir building dir/file lists for excluded directories, this was causing slow downs in crawling for directories excluded that contain a large amount of files/dirs
