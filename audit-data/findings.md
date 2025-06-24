### [H-1] Storing the password on-chain makes it visible to anyone, and is no longer private

**Description:** All data stored on-chain is publicly visible and can be read directly from the blockchain. The `PasswordStore::s_password` variable is intended to be private and should only be accessible through the `PasswordStore::getPassword` function.

Below we demonstrate one method of reading this data from the blockchain.

**Impact:** Anyone can read the private password, severely compromising the protocol's functionality.

**Proof of Concept:**

The following test case demonstrates how anyone can read the password directly from the blockchain:

1. Create a locally running chain:

```bash
make anvil
```

2. Deploy the contract:

```bash
make deploy
```

3. Run the storage tool (using slot `1` as it corresponds to `s_password` in the `PasswordStore` contract):

```bash
cast storage <CONTRACT_ADDRESS> 1 --rpc-url http://127.0.0.1:8545
```

This will output something like:

```
0x6d7950617373776f726400000000000000000000000000000000000000000014
```

4. Parse the hex output to a string:

```bash
cast parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014
```

Which will return:

```
myPassword
```

**Recommended Mitigation:** The contract architecture should be reconsidered. One approach would be to encrypt the password off-chain and store only the encrypted version on-chain. This would require users to remember a separate decryption key. Additionally, consider removing the view function to prevent accidental exposure of the decryption key in transaction data.

---

## Likelihood & Impact:

- Impact: HIGH

- Likelihood: HIGH

- Severity: HIGH

## High

- Worst offenders -> Least bad

## Medium

## Low

### [H-2] `PasswordStore::setPassword` has no access controls, allowing non-owners to change the password

**Description:** The `PasswordStore::setPassword` function is marked as `external` without any access controls, despite the NatSpec comment stating: "This function allows only the owner to set a new password."

```solidity
function setPassword(string memory newPassword) external {
    // @Audit - No access controls implemented
    s_password = newPassword;
    emit SetNewPassword();
}
```

**Impact:** Any user can set or change the stored password, completely undermining the contract's intended functionality.

**Proof of Concept:**

Add the following test to `PasswordStore.t.sol`:

<details>
<summary>Code</summary>

```solidity
function test_anyone_can_set_password(address randomAddress) public {
    vm.assume(randomAddress != owner);
    vm.startPrank(randomAddress);
    string memory expectedPassword = "myNewPassword";
    passwordStore.setPassword(expectedPassword);

    vm.startPrank(owner);
    string memory actualPassword = passwordStore.getPassword();
    assertEq(actualPassword, expectedPassword);
}
```

</details>

**Recommended Mitigation:** Add access control checks to the function:

```solidity
if(msg.sender != s_owner) {
    revert PasswordStore__NotOwner();
}
```

---

## Likelihood & Impact:

- Impact: HIGH

- Likelihood: HIGH

- Severity: HIGH

### [I-1] Incorrect NatSpec in `PasswordStore::getPassword` references non-existent parameter

**Description:** The NatSpec documentation for `PasswordStore::getPassword` incorrectly references a `newPassword` parameter that doesn't exist in the function signature.

```solidity
/*
 * @notice This allows only the owner to retrieve the password.
 * @param newPassword The new password to set.
 */
function getPassword() external view returns (string memory) {}
```

**Impact:** The documentation is misleading and incorrect.

**Recommended Mitigation:** Remove the incorrect parameter documentation:

```diff
/*
 * @notice This allows only the owner to retrieve the password.
- * @param newPassword The new password to set.
 */
```

### Likelihood & Impact

- Impact: NONE

- Likelihood: HIGH

- Severity: Informational/Gas/Non-crit

Informational: Hey, this isn't a bug, but you should know.
