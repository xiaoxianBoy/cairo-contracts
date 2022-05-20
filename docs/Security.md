# Security

 The following documentation provides context, reasoning, and examples of methods and constants found in `openzeppelin/security/`.

 > Expect this module to evolve.

## Table of Contents

* [Initializable](#initializable)
* [Pausable](#pausable)
* [Reentrancy Guard](#Reentrancy-Guard)

## Initializable

The Initializable library provides a simple mechanism that mimics the functionality of a constructor. More specifically, it enables logic to be performed once and only once which is useful to setup a contract's initial state when a constructor cannot be used.

The recommended pattern with Initializable is to include a check that the Initializable state is `False` and invoke `initialize` in the target function like this:

```cairo
from openzeppelin.security.initializable import Initializable

@external
func foo{
        syscall_ptr : felt*, 
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }():
    let (initialized) = Initializable.initialized()
    assert initialized = FALSE

    Initializable.initialize()
    return ()
end
```

> Please note that this Initializable pattern should only be used on one function.

## Pausable

The Pausable library allows contracts to implement an emergency stop mechanism. This can be useful for scenarios such as preventing trades until the end of an evaluation period or having an emergency switch to freeze all transactions in the event of a large bug.

To use the Pausable library, the contract should include `pause` and `unpause` functions (which should be protected). For methods that should be available only when not paused, insert `assert_not_paused`. For methods that should be available only when paused, insert `assert_paused`. For example:

```cairo
from openzeppelin.security.pausable import Pausable

@external
func whenNotPaused{
        syscall_ptr: felt*,
        pedersen_ptr: HashBuiltin*,
        range_check_ptr
    }():
    Pausable.assert_not_paused()

    # function body
    return ()
end

@external
func whenPaused{
        syscall_ptr: felt*,
        pedersen_ptr: HashBuiltin*,
        range_check_ptr
    }():
    Pausable.assert_paused()

    # function body
    return ()
end
```

> Note that `pause` and `unpause` already include these assertions. In other words, `pause` cannot be invoked when already paused and vice versa.

For a list of full implementations utilizing the Pausable library, see:

* [ERC20_Pausable](../src/openzeppelin/token/erc20/ERC20_Pausable.cairo)
* [ERC721_Mintable_Pausable](../src/openzeppelin/token/erc721/ERC721_Mintable_Pausable.cairo)

## Reentrancy Guard

A [reentrancy attack](https://gus-tavo-guim.medium.com/reentrancy-attack-on-smart-contracts-how-to-identify-the-exploitable-and-an-example-of-an-attack-4470a2d8dfe4) occurs when the caller is able to obtain more resources than allowed by recursively calling a target’s function.

Since Cairo does not support modifiers like Solidity, the [`reentrancy_guard`](../src/openzeppelin/security/reentrancy_guard.cairo) library exposes two methods `_start` and `_end` to protect functions against reentrancy attacks. The protected function must call `ReentrancyGuard._start` before the first function statement, and `ReentrancyGuard._end` before the return statement, as shown below:

```cairo
from openzeppelin.security.reentrancy_guard import ReentrancyGuard

func test_function{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }():
   ReentrancyGuard._start()
   # function body
   ReentrancyGuard._end()
   return ()
end
```