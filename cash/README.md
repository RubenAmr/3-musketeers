# ﻿💵Cash💵

## How to use the Cash App?

### 🏃‍♀️ Steps to do



- We first need to understand first the library features

```sh
❯ cd /path/to/workspace/3-musketeers/cash
❯ npm i
❯ node bin/index.js

# chech what's going on
# ...
```
#### _Constant.js_ file creation

- import the API _exchangeratesapi_ and some default currencies (you can add others if you want 😊 )
- Then, we export the modules to be able to use them in other code pages

```javascript

const API = 'https://api.exchangeratesapi.io/latest';
const DEFAULT_TO_CURRENCIES = ['USD', 'EUR', 'GBP', 'JPY'];

module.exports = {
  API,
  DEFAULT_TO_CURRENCIES
}
```



#### _Cash.js_ file creation

- You need to import the libraries you need and the _constant.js_ file link : 

```javascript
'use strict';

const got = require('got');
const money = require('money');
const chalk = require('chalk');
const ora = require('ora');
const currencies = require('../lib/currencies.json');
```



   - Then you can use the following function to convert a dollar into other currencies : 

```javascript
const cash = async command => {
	const {amount} = command;
	const from = command.from.toUpperCase();
	const to = command.to.filter(item => item !== from).map(item => item.toUpperCase());
/**
* We create a loading constant which will define how the user will wait for the result of the function and its format
**/
	console.log();
	const loading = ora({
		text: 'Converting...',
		color: 'green',
		spinner: {
			interval: 150,
			frames: to
		}
	});

	loading.start();
/*
* We import the API
* We tell the function to convert the currency in parameter with the other currencies in the array from Constant.js

*/
	await got(API, {
		json: true
	}).then(response => {
		money.base = response.body.base;
		money.rates = response.body.rates;

		to.forEach(item => {
			if (currencies[item]) {
				loading.succeed(`${chalk.green(money.convert(amount, {from, to: item}).toFixed(3))} ${`(${item})`} ${currencies[item]}`);
			} else {
				loading.warn(`${chalk.yellow(`The "${item}" currency not found `)}`);
			}
		});

		console.log(chalk.underline.gray(`\nConversion of ${chalk.bold(from)} ${chalk.bold(amount)}`));
	}).catch(error => {
		if (error.code === 'ENOTFOUND') {
			loading.fail(chalk.red('Please check your internet connection!\n'));
		} else {
			loading.fail(chalk.red(`Internal server error :(\n${error}`));
		}
		process.exit(1);
	});
};

```

#### _Index.js_ file creation

- In this file, we call the function we created in _Cash.js_ file and we select the configuration we want : 

  

  ```javascript
  #!/usr/bin/env node
  
  'use strict';
  
  const Conf = require('conf');
  const meow = require('meow');
  const chalk = require('chalk');
  const cash = require('./cash.js');
  
  const config = new Conf();
  const argv = process.argv.slice(2);
  
  const {DEFAULT_TO_CURRENCIES} = require('./constants');
  
  const cli = meow(`
  	Usage
  		$ cash <amount> <from> <to>
  		$ cash <options>
  	Options
  		--set -s 			Set default currencies
  	Examples
  		$ cash 10 usd eur pln
  		$ cash --set usd aud
  `);
  
  if (argv.indexOf('--save') !== -1 || argv.indexOf('-s') !== -1) {
  	config.set('defaultFrom', argv[1] || config.get('defaultFrom', 'USD'));
  	config.set('defaultTo', (argv.length > 2) ? process.argv.slice(4) : config.get('defaultTo', DEFAULT_TO_CURRENCIES));
  	console.log(chalk.green('Saved default currencies to ' + config.path));
  	process.exit(0);
  }
  
  const command = {
  	amount: parseFloat(argv[0]) || 1,
  	from: argv[1] || config.get('defaultFrom', 'USD'),
  	to: (argv.length > 2) ? process.argv.slice(4) : config.get('defaultTo', DEFAULT_TO_CURRENCIES)
  };
  
  cash(command);
  
  ```

- We should get this when we execute it : 

  ![](C:\Users\Ruben\Desktop\ESILV\Annee_4\Semester 2\Javascript\3-musketeers\cash\indexresult.PNG)

