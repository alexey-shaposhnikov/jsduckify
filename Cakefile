fs = require('fs')
path = require('path')
{spawn, exec} = require('child_process')
runsync = require('runsync')  # polyfil for node.js 0.12 synchronous running functionality. Remove when upgrading to 0.12
marked = require('marked')
wrench = require('wrench')

runAsync = (command, options, next) ->
  if options? and options.length > 0
    command += ' ' + options.join(' ')
  exec(command, (error, stdout, stderr) ->
    if stderr.length > 0 and error?
      console.log("Stderr exec'ing command '#{command}'...\n" + stderr)
      console.log('exec error: ' + error)
    if next?
      next(stdout)
    else
      if stdout.length > 0 and stdout?
        console.log("Stdout exec'ing command '#{command}'...\n" + stdout)
  )

runSync = (command, options, next) ->
  if options? and options.length > 0
    command += ' ' + options.join(' ')

  output = runsync.popen(command)
  stdout = output.stdout.toString()
  stderr = output.stderr.toString()
  if stderr.length > 0
    console.error("Error running `#{command}`\n" + stderr)
    process.exit(1)
  if next?
    next(stdout)
  else
    if stdout.length > 0
      console.log("Stdout exec'ing command '#{command}'...\n" + stdout)

task('docs', 'Generate docs ./docs', () ->
  process.chdir(__dirname)
  # create README.html
  readmeDotCSSString = fs.readFileSync('read-me.css', 'utf8')
  readmeDotMDString = fs.readFileSync('README.md', 'utf8')
  readmeDotHTMLString = marked(readmeDotMDString)
  readmeDotHTMLString = """
    <style>
    #{readmeDotCSSString}
    </style>
    <body>
    <div class="readme">
    #{readmeDotHTMLString}
    </div>
    </body>
  """
  fs.writeFileSync(path.join(__dirname, 'docs', 'README.html'), readmeDotHTMLString)

  # jsduckify
  {name, version} = require('./package.json')
  outputDirectory = path.join(__dirname, 'docs', "#{name}-docs")
  if fs.existsSync(outputDirectory)
    wrench.rmdirSyncRecursive(outputDirectory, false)
  runSync('bin/jsduckify', ['-d', "'" + outputDirectory + "'", "'" + __dirname + "'"])
)

task('pub-docs', 'Push docs to gh-pages on github', () ->
  pubDocsRaw()
)

pubDocsRaw = () ->
  process.chdir(__dirname)
  runAsync('git push -f origin master:gh-pages')

task('publish', 'Publish to npm and add git tags', () ->
  process.chdir(__dirname)
  runSync('cake test')  # Doing this exernally to make it synchrous
  invoke('docs')
  runSync('git status --porcelain', [], (stdout) ->
    if stdout.length == 0
      {stdout, stderr} = execSync('git rev-parse origin/master', true)
      stdoutOrigin = stdout
      {stdout, stderr} = execSync('git rev-parse master', true)
      stdoutMaster = stdout
      if stdoutOrigin == stdoutMaster
        console.log('running npm publish')
        {stdout, stderr} = execSync('npm publish .', true)
        if fs.existsSync('npm-debug.log')
          console.error('`npm publish` failed. See npm-debug.log for details.')
        else
          console.log('running pubDocsRaw()')
          pubDocsRaw()
          console.log('running git tag')
          runSync("git tag v#{require('./package.json').version}")
          runAsync("git push --tags")
      else
        console.error('Origin and master out of sync. Not publishing.')
    else
      console.error('`git status --porcelain` was not clean. Not publishing.')
  )
)

task('test', 'Run the CoffeeScript test suite with nodeunit', () ->
  {reporters} = require('nodeunit')
  process.chdir(__dirname)
  reporters.default.run(['test'], undefined, (failure) ->
    if failure?
      console.log(failure)
      process.exit(1)
  )
)
