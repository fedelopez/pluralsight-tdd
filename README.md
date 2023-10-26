# PluralSight Test Driven Development

## Agenda

[Class Slides](https://docs.google.com/presentation/d/1BKENiY7N3ZHVmZKxIbYqo0iI_SHMG7sl6_4wQ1pMVVY/edit?usp=sharing)

## Setup

Clone the repo from Github:

`git clone git@github.com:fedelopez/pluralsight-tdd.git && cd pluralsight-tdd`

Install the project dependencies:

`npm install`

A word about Jest: Jest is a JavaScript testing framework maintained by Facebook that works with a multitude of front-end frameworks and can even
be used to test node apps. It is specially well suited to test React apps. 

Create a file named `bank.test.js` under the `src` folder

And paste the following:

```js
describe('Bank', function () {
    it('should pass', function () {
        expect(1).toBe(1)
    });
});
```

A note on `describe` and `it`: `it` simply describes the suite of test cases enumerated by the `it` functions.
Each `it` function is a test. Each `describe` function contains several `it` tests.

Run the test from the command using `npm test` to ensure the dependencies work well. The test should be successful. :

```text
$ jest
 PASS  ./bank.test.js
  Bank
    ✓ should pass (3ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        0.942s, estimated 1s
Ran all test suites.
✨  Done in 1.68s.
```

You are all set, let's learn about TDD!

### Part I: TDDing the logic

### Testing the bank initialisation

Replace the first test (which adds no value) with the following:

```js
import {Bank} from './bank';

it('creates a new bank with no accounts', function () {
    const bank = new Bank();
    expect(bank.accounts.length).toBe(0)
});
```

#### Calling the shot

Before running the test say out loud what’s going to happen; how is the test going to fail?

Calling the shot is the only way we know we have written a good test. 
If the test fails for another reason it means that there is something wrong with the setup. 
If it passes when we expected it to fail, something is wrong. 

Calling the shot is how we test the test itself.

Read [TDD: Call your shots](https://markhneedham.com/blog/2010/07/28/tdd-call-your-shots)

#### Implementing the bank initialisation

After running the test we get this error message:

```bash
Error: Cannot find module './bank.js' from 'bank.test.js'
``` 

Is this what you expected?

Let's fix our first test.

Create the file `bank.js`.

Now implement the getter method `accounts` by just returning an empty array:

```js
export class Bank {
    get accounts() {
        return [];
    };
}
```

The code above is just enough to make the test pass. 

### Adding an account

Let's add a to add an account to the bank:

```js
it('adds a new account', function () {
    const bank = new Bank();
    
    const accountName = 'savings';
    bank.addAccount(accountName);
    
    const accounts = bank.accounts;
    expect(accounts.length).toBe(1);
    expect(accounts[0].accountName).toBe(accountName);
});
```

The bank class now has a new method `addAccount`. Each account is now tracked in a private field named `accounts`:

```js
class Bank {
     #accounts = [];
    
     get accounts() {
        return this.#accounts;
     }

    addAccount(accountName) {
        this.accounts.push({accountName: accountName});
    }
}
```

Running `npm test` shows both tests successful:

```text
±  |master S:3 U:2 ?:5 ✗| → yarn test
yarn run v1.17.3
$ jest
 PASS  ./bank.test.js
  Bank
    ✓ creates a new bank with no accounts (27ms)
    ✓ adds a new account (1ms)

Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        1.495s, estimated 2s
Ran all test suites.
✨  Done in 2.13s.
```

### Refactoring

We have duplicated some code in our test class, can you tell what is it?

In TDD test classes are first-class citizens, and we take as much care of them as with the production classes.

Let's remove the duplication of the bank creation by introducing a `beforeEach`:

```js
import {Bank} from './bank';

describe('Bank', function () {
    let bank; // don't initialise here, use a beforeEach instead
    
    beforeEach(function () {
        bank = new Bank();
    });

    it('creates a new bank with no accounts', function () {
        expect(bank.accounts.length).toBe(0)
    });

    it('adds a new account', function () {
        const accountName = 'savings';
        bank.addAccount(accountName);

        const accounts = bank.accounts;
        expect(accounts.length).toBe(1);
        expect(accounts[0].accountName).toBe(accountName);
    });
});
``` 

### Deposit funds into an account

```js
it('deposits funds in an existing account', function () {
    const accountName = 'cheque';
    const amount = 42;
    bank.addAccount(accountName);
    bank.deposit(accountName, amount);

    const accounts = bank.accounts();
    expect(accounts[0].accountName).toBe('cheque');
    expect(accounts[0].amount).toBe(amount);
});
```

It seems we have done our job, but we get this error:

```bash
Expected: 42
Received: NaN
```

We need to update the creation of the account to initialise the default amount to 0. 

```js
addAccount(accountName) {
    const account = {accountName: accountName, amount: 0};
    accounts.push(account);
}
```

And now let's add a method that deposits the funds on the correct bank account:

```js
deposit(accountName, amount) {
    const account = accounts.find(function (account) {
        return account.accountName === accountName;
    });
    account.amount = account.amount + amount;
}
```

Refactor: We can introduce an `Account` class as part of our DSL.

```js
addAccount(accountName) {
    const account = new Account(accountName, 0);
    accounts.push(account);
}

// and the new class
export class Account {
    #accountName;
    #amount;

    constructor(accountName, amount) {
        this.#accountName = accountName;
        this.#amount = amount;
    }

    get accountName() {
        return this.#accountName;
    }

    get amount() {
        return this.#amount;
    }

    set accountName(value) {
        this.#accountName = value;
    }

    set amount(value) {
        this.#amount = value;
    }
}
```


### Withdraw from an account

```js
it('withdraws funds from an existing account', function () {
    const accountName = 'credit';
    const amount = 100;
    bank.addAccount(accountName);
    bank.deposit(accountName, amount);
    bank.withdraw(accountName, 73);

    const accounts = bank.accounts();
    expect(accounts[0].amount).toBe(27);
});
```

Similarly to `deposit`, we have to find the account and decrement the funds:

```js
withdraw(accountName, amount) {
    const account = accounts.find(function (account) {
        return account.accountName === accountName;
    });
    account.amount = account.amount - amount;
}
```

#### Refactoring

Both methods `deposit` and `withdraw` have to find the desired account. Let's refactor that duplication:

Step 1: create a `findAccount` method inside Bank: 

```js
findAccount(accountName) {
    return accounts.find(function (account) {
        return account.accountName === accountName;
    });
}
```

Step 2: use the newly created method in `deposit` and `withdraw`:

```js
deposit(accountName, amount) {
    const account = findAccount(accountName);
    account.amount = account.amount + amount;
}

withdraw(accountName, amount) {
    const account = findAccount(accountName);
    account.amount = account.amount - amount;
}
```

#### Next steps

What edge cases we are not testing here?

