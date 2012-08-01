fs            = require 'fs'
{print, pump} = require 'util'
{spawn, exec} = require 'child_process'

# ANSI Terminal Colors
bold  = '\x1B[0;1m'
red   = '\x1B[0;31m'
green = '\x1B[0;32m'
reset = '\x1B[0m'

pkg = JSON.parse fs.readFileSync('./src/package.json')
testCmd = pkg.scripts.test
startCmd = pkg.scripts.start

log = (message, color, explanation) ->
  console.log color + message + reset + ' ' + (explanation or '')

# Compiles app.coffee and src directory to the app directory
copy = (src, dst, callback) ->
  pump fs.createReadStream(src), fs.createWriteStream(dst), callback

build = (callback) ->
  copy './src/package.json', './bin/package.json', (err) ->
    options = ['-c', '-b', '-o', 'bin', 'src']
    coffee = spawn 'coffee', options
    coffee.stdout.pipe process.stdout
    coffee.stderr.pipe process.stderr
    coffee.on 'exit', (status) -> callback?() if status is 0

# mocha test
test = (callback) ->
  options = [
    '--reporter'
    'spec'
    '--compilers'
    'coffee:coffee-script'
    '--colors'
    '--require'
    'assert'
    '--require'
    'should'
  ]
  spec = spawn 'mocha', options
  spec.stdout.pipe process.stdout 
  spec.stderr.pipe process.stderr
  spec.on 'exit', (status) -> callback?() if status is 0

install = (dep, callback) ->
  options = [ 'install', dep ]
  npm = spawn 'npm', options
  npm.stdout.pipe process.stdout
  npm.stderr.pipe process.stdout
  npm.on 'exit', (status) -> callback?() if status is 0

task 'deps', ->
  deps = (( dep for dep of pkg.dependencies ).concat ( dep for dep of pkg.devDependencies ))
  callback = ->
    dep = deps.shift()
    install dep, callback if dep?
  callback()

task 'docs', 'Generate annotated source code with Docco', ->
  fs.readdir 'src', (err, contents) ->
    files = ("src/#{file}" for file in contents when /\.coffee$/.test file)
    docco = spawn 'docco', files
    # docco.pipe process.stdout
    docco.stdout.pipe process.stdout
    docco.stderr.pipe process.stderr
    docco.on 'exit', (status) -> callback?() if status is 0

task 'spec', 'Run Mocha tests', ->
  build -> test -> log ":)", green

task 'test', 'Run Mocha tests', ->
  build -> test -> log ":)", green

task 'build', ->
  build -> log ":)", green

task 'dev', 'start dev env', ->
  # watch_coffee
  options = ['-c', '-b', '-w', '-o', 'bin', 'src']
  coffee = spawn 'coffee', options
  coffee.stdout.pipe process.stdout
  coffee.stderr.pipe process.stderr
  log 'Watching coffee files', green
