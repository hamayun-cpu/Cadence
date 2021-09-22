// Declare a resource interface for a fungible token.
// Only resources can implement this resource interface.
//
pub resource interface FungibleToken {

    // Require the implementing type to provide a field for the balance
    // that is readable in all scopes (`pub`).
    //
    // Neither the `var` keyword, nor the `let` keyword is used,
    // so the field may be implemented as either a variable field,
    // a constant field, or a synthetic field.
    //
    // The read balance must always be positive.
    //
    // NOTE: no requirement is made for the kind of field,
    // it can be either variable or constant in the implementation.
    //
    pub balance: Int {
        set(newBalance) {
            pre {
                newBalance >= 0:
                    "Balances are always set as non-negative numbers"
            }
        }
    }

    // Require the implementing type to provide an initializer that
    // given the initial balance, must initialize the balance field.
    //
    init(balance: Int) {
        pre {
            balance >= 0:
                "Balances are always non-negative"
        }
        post {
            self.balance == balance:
                "the balance must be initialized to the initial balance"
        }

        // NOTE: The declaration contains no implementation code.
    }

    // Require the implementing type to provide a function that is
    // callable in all scopes, which withdraws an amount from
    // this fungible token and returns the withdrawn amount as
    // a new fungible token.
    //
    // The given amount must be positive and the function implementation
    // must add the amount to the balance.
    //
    // The function must return a new fungible token.
    // The type `{FungibleToken}` is the type of any resource
    // that implements the resource interface `FungibleToken`.
    //
    pub fun withdraw(amount: Int): @{FungibleToken} {
        pre {
            amount > 0:
                "the amount must be positive"
            amount <= self.balance:
                "insufficient funds: the amount must be smaller or equal to the balance"
        }
        post {
            self.balance == before(self.balance) - amount:
                "the amount must be deducted from the balance"
        }

        // NOTE: The declaration contains no implementation code.
    }

    // Require the implementing type to provide a function that is
    // callable in all scopes, which deposits a fungible token
    // into this fungible token.
    //
    // No precondition is required to check the given token's balance
    // is positive, as this condition is already ensured by
    // the field requirement.
    //
    // The parameter type `{FungibleToken}` is the type of any resource
    // that implements the resource interface `FungibleToken`.
    //
    pub fun deposit(_ token: @{FungibleToken}) {
        post {
            self.balance == before(self.balance) + token.balance:
                "the amount must be added to the balance"
        }

        // NOTE: The declaration contains no implementation code.
    }

}

// Declare a resource named `ExampleToken` that has to implement
// the `FungibleToken` interface.
//
// It has a variable field named `balance`, that can be written
// by functions of the type, but outer scopes can only read it.
//
pub resource ExampleToken: FungibleToken {

    // Implement the required field `balance` for the `FungibleToken` interface.
    // The interface does not specify if the field must be variable, constant,
    // so in order for this type (`ExampleToken`) to be able to write to the field,
    // but limit outer scopes to only read from the field, it is declared variable,
    // and only has public access (non-settable).
    //
    pub var balance: Int

    // Implement the required initializer for the `FungibleToken` interface:
    // accept an initial balance and initialize the `balance` field.
    //
    // This implementation satisfies the required postcondition.
    //
    // NOTE: the postcondition declared in the interface
    // does not have to be repeated here in the implementation.
    //
    init(balance: Int) {
        self.balance = balance
    }

    // Implement the required function named `withdraw` of the interface
    // `FungibleToken`, that withdraws an amount from the token's balance.
    //
    // The function must be public.
    //
    // This implementation satisfies the required postcondition.
    //
    // NOTE: neither the precondition nor the postcondition declared
    // in the interface have to be repeated here in the implementation.
    //
    pub fun withdraw(amount: Int): @ExampleToken {
        self.balance = self.balance - amount
        return create ExampleToken(balance: amount)
    }

    // Implement the required function named `deposit` of the interface
    // `FungibleToken`, that deposits the amount from the given token
    // to this token.
    //
    // The function must be public.
    //
    // NOTE: the type of the parameter is `{FungibleToken}`,
    // i.e., any resource that implements the resource interface `FungibleToken`,
    // so any other token – however, we want to ensure that only tokens
    // of the same type can be deposited.
    //
    // This implementation satisfies the required postconditions.
    //
    // NOTE: neither the precondition nor the postcondition declared
    // in the interface have to be repeated here in the implementation.
    //
    pub fun deposit(_ token: @{FungibleToken}) {
        if let exampleToken <- token as? ExampleToken {
            self.balance = self.balance + exampleToken.balance
            destroy exampleToken
        } else {
            panic("cannot deposit token which is not an example token")
        }
    }

}

// Declare a constant which has type `ExampleToken`,
// and is initialized with such an example token.
//
let token <- create ExampleToken(balance: 100)

// Withdraw 10 units from the token.
//
// The amount satisfies the precondition of the `withdraw` function
// in the `FungibleToken` interface.
//
// Invoking a function of a resource does not destroy the resource,
// so the resource `token` is still valid after the call of `withdraw`.
//
let withdrawn <- token.withdraw(amount: 10)

// The postcondition of the `withdraw` function in the `FungibleToken`
// interface ensured the balance field of the token was updated properly.
//
// `token.balance` is `90`
// `withdrawn.balance` is `10`

// Deposit the withdrawn token into another one.
let receiver: @ExampleToken <- // ...
receiver.deposit(<-withdrawn)
