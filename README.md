# Parallel Testing on Github?  Yes Please!
![Sauce Labs Tests](https://github.com/DylanLacey/ParallelRSpecAction/workflows/Sauce%20Labs%20Tests/badge.svg?event=push)
This is an example of a quality gated workflow for running parallel tests on Sauce Labs from Github Actions.

It uses the parallel_tests gem and RSpec as examples, but the workflow is applicable to most languages and tools.

Not Included: Selenium setup for Sauce Labs; This workflow assumes your tests are already running against Sauce.

## Using this action in your own tests
### You Will Need:
* An active Sauce Labs account (Or the [free trial](https://saucelabs.com/sign-up) why not?)
* A Github repo set up with your Ruby code
* Your app to run externally to Github, as this demo doesn't support Sauce Connect (for now)

### Then do this
1. Set up your [sneaky secret](https://docs.github.com/en/free-pro-team@latest/actions/reference/encrypted-secrets) `SAUCE_USERNAME` and `SAUCE_ACCESS_KEY`  
2. Make sure your tests get their desired capabilities from the `browserName`, `browserVersion` and `platformName`
3. Add `.github/workflows/saucelabs.yaml` to your own `.github/workflows` directory
4. Edit the defaults under `quality_gate.env` to match your needs
4. Add the parallel_tests gem to your Gemfile and make sure `parallel_rspec` does what you want
5. Add the `.parallel_rspec` file to your root directory

That should pretty much be it!

## What the workflow does
### Runs Tests
Well Duh Ned, with what?

### With as much Sauce Labs concurrency as it can get, for a single browser.  At first.
The workflow checks out your code, installs dependencies, then runs the `parallel_tests` RSpec runner.  This kicks off n parallel tests in the same Browser configuration, where n is 5 by default.

_I recommend changing the default to match your Sauce Labs concurrency limit; The faster you run all your tests on a single platform, the better_

Default values for the are set under `jobs.quality_gate.env`. As long as your desired capabilities are taken from these variables, changing them will change what platform your tests run on.  EG to change the default browser to Firefox, edit `browserName` thusly:

```yaml
      browserName: ${{ github.event.inputs.browserName || 'Firefox' }}
```

### So my tests pass once, what then?
Once your entire test suite has passed for a single browser (thus proving your code works), your suite is run again, once for every session configured in `browser_coverage.strategy.matrix`.  See (Using a build matrix)[https://docs.github.com/en/free-pro-team@latest/actions/learn-github-actions/managing-complex-workflows#using-a-build-matrix] for details on how that works.  These tests also run in parallel, one for every available concurrency.

This proves that the individual browsers work with your code.  It's a much slower process, but the rapid feeback from the first phase lets you get back to work and let browser tests chug along in the background

### Caching
This setup caches the installed gems, making it quicker to set up your code on subsequent runs.

### Test Time Management
By relying on the parallel_tests gem's time tracking feature, tests are divided up during the `quality_gate` step to make all tests finish as rapidly as possible.

### Status Badges.
Easily done!  (See here)[https://docs.github.com/en/free-pro-team@latest/actions/managing-workflow-runs/adding-a-workflow-status-badge]

### Manual Excution for specific platforms
Because this workflow has a `workflow_dispatch` trigger, you're able to manually trigger a run from the Actions tab.  When doing so, you can pick the platform you want to test against, as well as the amount of concurrency to test with.