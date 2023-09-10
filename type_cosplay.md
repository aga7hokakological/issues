In solidity type cosplay can be considered as access control issue where a function is supposed to be called by particular user/owner but because access control is missing it can be called by malicious to behave it in uninteded way.
In solana account data is stored as an array of bytes which is transacted and then program deserializes it into account type. Here type cosplay means that if there are no conditions to differntiate between the account then it is possible that
any unexpected account might be used in the place of the expected account. This kind of issue can lead to unexpected results.

In rust we can use account discriminator to differntiate the account types:
```rust
#[derive(BorshSerialize, BorshDeserialize)]
pub struct Owner {
    discriminant: AccountDiscriminant,
    owner: Pubkey,
}

#[derive(BorshSerialize, BorshDeserialize, PartialEq)]
pub enum AccountDiscriminant {
    User,
    Owner,
}
```
```rust
if owner.discriminant != AccountDiscriminant::Owner {
    return Err(ProgramError::InvalidAccountData.into());
}
```

In the following code you can see that the User and Metadata has same structure. It means that the Metadata account can be pass as User account and it'll update the user instead of updating real user.
The transaction will pass as long as the authority of the account matches who is going to sign the transaction.
```rust
use anchor_lang::prelude::*;
use borsh::{BorshDeserialize, BorshSerialize};

declare_id!("Fg6PaFpoGXkYsidMpWTK6W2BeZ7FEfcYkg476zPFsLnS");

#[program]
pub mod type_cosplay_insecure {
    use super::*;

    pub fn update_user(ctx: Context<UpdateUser>) -> ProgramResult {
        let user = User::try_from_slice(&ctx.accounts.user.data.borrow()).unwrap();
        if ctx.accounts.user.owner != ctx.program_id {
            return Err(ProgramError::IllegalOwner);
        }
        if user.authority != ctx.accounts.authority.key() {
            return Err(ProgramError::InvalidAccountData);
        }
        msg!("GM {}", user.authority);
        Ok(())
    }
}

#[derive(Accounts)]
pub struct UpdateUser<'info> {
    user: AccountInfo<'info>,
    authority: Signer<'info>,
}

#[derive(BorshSerialize, BorshDeserialize)]
pub struct User {
    authority: Pubkey,
}

#[derive(BorshSerialize, BorshDeserialize)]
pub struct Metadata {
    account: Pubkey,
}
```
To resolve the issue as suggested in rust code we can implement the account discriminator in anchor.
So now when the user is to be updated we need to just add the check that account data is getting updated instead of the metadata account.
```rust
if user.discriminant != AccountDiscriminant::User {
            return Err(ProgramError::InvalidAccountData);
        }
```
Following is the full code which checks the user account:
```rust
use anchor_lang::prelude::*;
use borsh::{BorshDeserialize, BorshSerialize};

declare_id!("Fg6PaFpoGXkYsidMpWTK6W2BeZ7FEfcYkg476zPFsLnS");

#[program]
pub mod type_cosplay_secure {
    use super::*;

    pub fn update_user(ctx: Context<UpdateUser>) -> ProgramResult {
        let user = User::try_from_slice(&ctx.accounts.user.data.borrow()).unwrap();
        if ctx.accounts.user.owner != ctx.program_id {
            return Err(ProgramError::IllegalOwner);
        }
        if user.authority != ctx.accounts.authority.key() {
            return Err(ProgramError::InvalidAccountData);
        }
        if user.discriminant != AccountDiscriminant::User {
            return Err(ProgramError::InvalidAccountData);
        }
        msg!("GM {}", user.authority);
        Ok(())
    }
}

#[derive(Accounts)]
pub struct UpdateUser<'info> {
    user: AccountInfo<'info>,
    authority: Signer<'info>,
}

#[derive(BorshSerialize, BorshDeserialize)]
pub struct User {
    discriminant: AccountDiscriminant,
    authority: Pubkey,
}

#[derive(BorshSerialize, BorshDeserialize)]
pub struct Metadata {
    discriminant: AccountDiscriminant,
    account: Pubkey,
}

#[derive(BorshSerialize, BorshDeserialize, PartialEq)]
pub enum AccountDiscriminant {
    User,
    Metadata,
}
```

The other way to secure your program from type cosplay issue is to use the has_one constraint.
In Anchor, program account types automatically implement the Discriminator trait which creates an 8 byte unique identifier for a type.
Use Anchor’s Account<'info, T> type to automatically check the discriminator of the account when deserializing the account data
As you can see from the code when using #[account] on the account type it automatically checks the necessary validations.
```rust
use anchor_lang::prelude::*;
use borsh::{BorshDeserialize, BorshSerialize};

declare_id!("Fg6PaFpoGXkYsidMpWTK6W2BeZ7FEfcYkg476zPFsLnS");

#[program]
pub mod type_cosplay_recommended {
    use super::*;

    pub fn update_user(ctx: Context<UpdateUser>) -> ProgramResult {
        msg!("GM {}", ctx.accounts.user.authority);
        Ok(())
    }
}

#[derive(Accounts)]
pub struct UpdateUser<'info> {
    #[account(has_one = authority)]
    user: Account<'info, User>,
    authority: Signer<'info>,
}

#[account]
pub struct User {
    authority: Pubkey,
}

#[account]
pub struct Metadata {
    account: Pubkey,
}
```
