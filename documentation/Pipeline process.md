# Pipeline process Explaination

## Step one (Push code)
Push the new changes to GitHub 

```bash
git push
```

## Step two (ci/cd)
1. Circleci senses change only on the main branch and spin up the environment 

``` yaml 

      - hold:
          filters:
            branches:
              only:
                - main

```

1. Circleci Preparing environment variables 
1. Install Node.js 14.15 

``` yaml 
jobs:
    build:
        docker: 
            - image: "cimg/node:14.15"
        steps: 
        - node/install:
            node-version: '14.15'
```

1. Setting up Elastic Beanstalk CLI

``` yaml 
    - eb/setup
```


1. Install AWS CLI latest 

``` yaml 
    - aws-cli/setup
```

1. Configure AWS Access Key ID

1. Checkout code 

``` yaml
    - checkout
```

1. Install front-End Dependencies 

``` yaml
 - run:
          name: Install Front-End Dependencies
          command: |
            echo "NODE --version" 
            echo $(node --version)
            echo "NPM --version" 
            echo $(npm --version)
            npm run frontend:install

```


1. Install API Dependencies 

``` yaml 
 - run:
          name: Install API Dependencies
          command: |
           echo "NODE --version"
           echo $(node --version)
           echo "NPM --version"
           echo $(npm --version)
           npm run api:install
```

1. Front-end Build 

``` yaml 
- run:
          name: Front-End Build
          command: |
            echo "Build the frontend app"
            npm run frontend:build
```

1. EB config 

``` yaml 
 - run:
          name: EB config
          command: | 
            npm run api:eb

```

1. APi Build 

``` yaml 

   - run:
          name: API Build
          command: |
            echo "Build the backend API"
            npm run api:build
```

1. Circleci hold to approve the deployment

``` yaml 
workflows:
  udagram:
    jobs:
      - build
      - hold:
          filters:
            branches:
              only:
                - main
          type: approval
          requires:
            - build

```

1. after approval, Circleci start to deploy

``` yaml 
workflows:
  udagram:
    jobs:
      - build
      - hold:
          filters:
            branches:
              only:
                - main
          type: approval
          requires:
            - build
      - deploy:
          requires:
            - hold
```

1. Install Node.js 14.15 and Elastic Beanstalk CLI, AWS CLI Setup, and Checkout code 

``` yaml 
deploy:
    docker:
      - image: "cimg/base:stable"
      # more setup needed for aws, node, elastic beanstalk
    steps:
      - node/install:
          node-version: '14.15' 
      - eb/setup
      - aws-cli/setup
      - checkout

```

1. Deploy in both apps 

``` yaml 
- run:
    name: Deploy App
        # Install, build, deploy in both apps
    command: |
        echo "#  Install, build, deploy in both apps"
        npm run frontend:deploy
        npm run api:eb
        npm run api:build
        npm run api:deploy

```

