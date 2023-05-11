# Circleci
To automatically cancel a CircleCI pipeline deployment after a certain amount of time has passed, you can use CircleCI's built-in functionality for job timeouts and job-level custom logic. Here's an approach you can take:

1. Set a timeout for the deployment job(s) in your pipeline. You can do this by adding a `timeout` parameter to the job definition in your CircleCI configuration file. For example:

```
jobs:
  deploy:
    timeout: 10 hours
    steps:
      - run: deploy_to_production.sh
```

This will cause the `deploy` job to automatically fail if it runs for more than 10 hours.

2. Add custom logic to the deployment job(s) that checks if the pipeline should be cancelled. You can do this using a combination of environment variables and conditional logic in your script. For example:

```
jobs:
  deploy:
    timeout: 10 hours
    environment:
      CIRCLE_START_TIME: $CIRCLE_BUILD_TIME
    steps:
      - run: deploy_to_production.sh
      - run: |
          if [ $(($(date +%s) - $CIRCLE_START_TIME)) -gt 36000 ]; then
            echo "Pipeline has been running for more than 10 hours. Cancelling deployment."
            exit 1
          fi
```

This will set an environment variable `CIRCLE_START_TIME` to the build start time, and then check if the current time minus the start time is greater than 36,000 seconds (i.e. 10 hours). If so, it will print a message and exit with an error code, causing the job to fail.

3. Add a conditional step to the pipeline that cancels the deployment job(s) if they fail due to the timeout or custom logic. You can do this using CircleCI's `when` parameter. For example:

```
workflows:
  deploy:
    jobs:
      - deploy:
          when: on_success
      - cancel_deploy:
          when: on_fail
    # ...
  
jobs:
  # ...
  cancel_deploy:
    steps:
      - run: circleci cancel $CIRCLE_WORKFLOW_ID
```

This will add a `cancel_deploy` job to the pipeline that runs only if the `deploy` job succeeds. If the `deploy` job fails due to the timeout or custom logic, the `cancel_deploy` job will run and cancel the entire workflow using the CircleCI API.

By combining these steps, you can automatically cancel a CircleCI pipeline deployment if it runs for more than a specified amount of time without being deployed.
