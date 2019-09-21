# Quasar, QEnv, and Firebase hosting with a Production and Staging site

### Firebase

Create two projects: "ProjectName-Prod" and "ProjectName-Staging" -- I added analytics to Prod, but not Staging. If you are going to have clients previewing features in staging then you might setup analytics in both.


Get the config info for each project.

PRODUCTION

```
const firebaseConfig = {
  apiKey: "*************************",
  ...
  projectId: "projectname-prod",
  ...
}
```

STAGING

```
const firebaseConfig = {
  ...
  projectId: "projectame-staging",
  ...
}
```

### Add the QEnv App Extension

Qenv is more flexible then dotenv, if you have a client specifying dotenv this process flow should still work for you. But I think the spirit of Quasar is Best Practices Plus and that's what you get with QEnv ... star the repo. Install from within your quasar project with the following command:

`quasar ext add @quasar/qenv`

Now, during setup you might want to use a common name like "firebase" or your name or initals. I did use a common name here, opting instead to use objects inside my env config file -- but now that I see how everything fits and flows, I probably will in the next project. When the install is done running you will want to edit the `.quasar.env.json` inserting the firebase config info for your production and staging environments. Mine looks like this...

```
{
  "development": {
    "ENV_TYPE": "Running Development",
    "ENV_DEV": "Development"
  },
  "staging": {
    "ENV_TYPE": "Running Staging",
    "ENV_ID": "staging",
    "firebaseConfig": {
      "apiKey": "********",
      "authDomain": "*******-staging*********",
      "databaseURL": "*****url******",
      "projectId": "projectname-staging",
      "storageBucket": "********",
      "messagingSenderId": "********",
      "appId": "**************"
    }
  },
  "production": {
    "ENV_TYPE": "Running Production",
    "ENV_ID": "production",
    "firebaseConfig": {
      "apiKey": "********",
      "authDomain": "*******-prod*********",
      "databaseURL": "*****url******",
      "projectId": "projectname-prod",
      "storageBucket": "********",
      "messagingSenderId": "********",
      "appId": "**************"      
    }
  },
  "test": {
    "ENV_TYPE": "Running Test",
    "ENV_Test": "Test"
  }
}
```

### Add Firebase

Add firebase to the project with yarn or npm.

`yarn add firebase`

Then run firebase init. Note, I want staging to be the default, so I specified that when I ran init...

```
firebase init 
  Hosting
  select the FB project (myproject-staging) 
  directory dist/pwa  or dist/spa whichever you need or prefer
```

The initialization creates two files, `.firebaserc` & `firbase.json` and you should inspect their form.

Now is a good time to revisit our objective: we wish to conditionally deploy a single local project, to either a production or staging site hosted at firebase. This is a common ojbective and firebase has a command that you can leverage the `firebase use` command. You alias your sites, then you can use commands like `firebase use staging` and `firebase use production`.

```
~$ firebase use --add
? Which project do you want to add? projectname-prod
? What alias do you want to use for this project? (e.g. staging) production

Created alias production for projectname-prod.
Now using alias production (projectname-prod)
```

so .firebaserc looks like this now:

```
{
  "projects": {
    "default": "projectname-staging",
    "production": "projectame-prod"
  }
}
```

To get back to staging specify the default alias.

```
$ firebase use default
Now using alias default (projectname-staging)
```

Now you are going to edit and expand the scripts section of your `package.json` file...


### tweak the package.json scripts

So the process flow is for using quasar/qenv/fireabase is `fireabase use [specifalias]` then `QENV=environment` then `qauasar dev|build` then if you are deploying you add `firebase deploy`. This is what it looks like in my project, 'dev' and 'staging' are identical so that I don't need to recall if I named it one or the other, it will work with either `yarn dev` or `yarn staging`. 

```
  "scripts": {
  "private": true,
  "scripts": {
    "lint": "eslint --ext .js,.vue src",
    "test": "echo \"No test specified\" && exit 0",
    "dev": "firebase use default && QENV=staging quasar dev",
    "staging": "firebase use default && QENV=staging quasar dev",
    "deploy-to-staging": "firebase use default && QENV=staging quasar build && firebase deploy",
    "deploy-to-production": "firebase use production && QENV=production quasar build && firebase deploy"
    },
  },
  ```

So when it's time to update my staging site I will run `yarn deploy-to-staging` and `yarn deploy-to-production` when I wish to update the production site.



### Incorporate Firebase into Quasar

You want to create a bootfile, `src/boot/firebase.js file` so use the quasar cli tools.

```bash
quasar new boot firebase
```

When you set up the simplest Quasar/Firebase project, you put the Firebase Config information in a config file, import it into your boot file, and initialize Firebase. The change here, to accommodate QEnv and deployment to two or more Firebase sites is to read the config information from the process.env -- where it will be sitting thanks to your `QENV=[environment]` command in your `package.json`'s script section.

```
const firebaseConfig = {
  apiKey: process.env.firebaseConfig.apiKey,
  authDomain: process.env.firebaseConfig.authDomain,
  databaseURL: process.env.firebaseConfig.databaseURL,
  projectId: process.env.firebaseConfig.projectId,
  storageBucket: process.env.firebaseConfig.storageBucket,
  messagingSenderId: process.env.firebaseConfig.messagingSenderId
}
```

If you are using vuefire or other firebase or firestore helpers be sure to add them to `yarn add vuefire`.

now edit the quasar.conf.js and add it to the boot section

```
    boot: [
      'firebase'
    ],
```


### Check if it's working

To see if the setup was working, tweaked the Index.vue like this, and monitored the developer console to ensure that the proper values were being used.

```
<template>
  <q-page class="flex flex-center">
    <q-btn
      label="Env"
      @click="envTest"
    ></q-btn>
  </q-page>
</template>

<script>
export default {
  name: 'PageIndex',
  methods: {
    envTest () {
      console.log('Current Env: ', process.env)

      if (process.env.ENV_ID === 'staging') {
        console.log('using staging QENV')
        console.log(process.env.ENV_ID)
        console.log(process.env.firebaseConfig.projectId)
      }

      if (process.env.ENV_ID === 'production') {
        console.log(process.env.ENV_ID)
        console.log(process.env.firebaseConfig.projectId)
      }
    }

  }
}
</script>
```

`yarn dev`
 
open up the console, clear it, click the ENV button inspect results 
