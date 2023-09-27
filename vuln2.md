/*
    You know your password right? Are yoy logging in to correct software? What's the issue here?
*/ 

```rust
use anchor_lang::prelude::*;

declare_id!("5EdL7WTNNVxt1xkybts4TVn5QAhpNGkUS7sZqa7xF8xS");

#[program]
pub mod store_password {
    use super::*;

    pub fn initialize(ctx: Context<Initialize>) -> Result<()> {
        let user_account = &mut ctx.accounts.user_account;

        Ok(())
    }

    pub fn set_password(ctx: Context<SetPassword>, pass: String) -> Result<()> {
        let user_account = &mut ctx.accounts.user_account;
        user_account.password = pass;

        Ok(())
    }
}

#[derive(Accounts)]
pub struct Initialize<'info> {
    #[account(
        init,
        payer = user,
        space = 32 + 8
    )]
    pub user_account: Account<'info, User>,
    #[account(mut)]
    pub user: Signer<'info>,
    pub system_program: Program<'info, System>,
}

#[derive(Accounts)]
pub struct SetPassword<'info> {
    #[account(mut)]
    pub user_account: Account<'info, User>,
}

#[account]
pub struct User {
    pub password: String,
}
```
```rust
use anchor_lang::prelude::*;
use store_password::cpi::accounts::SetPassword;
use anchor_lang::solana_program::{instruction::Instruction, program::invoke};

declare_id!("HmbTLCmaGvZhKnn1Zfa1JVnp7vkMV4DYVxPLWBVoN65L");

#[program]
pub mod login {
    use super::*;

    pub fn login(ctx: Context<Login>, pass: String) -> Result<()> {
        let context = CpiContext::new(
            ctx.accounts.store_password_program.to_account_info(),
            SetPassword {
                user_account: ctx.accounts.user.to_owned(),
                user: ctx.accounts.user.to_account_info(),
                system_program: ctx.accounts.system_program.to_account_info(),
            },
        );

        let create_instruction = Instruction {
            program_id: context.program.key(),
            accounts: vec![
                AccountMeta::new(ctx.accounts.user_account.key(), false),
                AccountMeta::new(ctx.accounts.user.key(), true),
                AccountMeta::new_readonly(ctx.accounts.system_program.key(), false),
            ],
            data: pass,
        };

        invoke(
            &create_instruction,
            &context.accounts.to_account_infos(),
        )?;

        Ok(())
    }
}

#[derive(Accounts)]
pub struct Login<'info> {
    #[account(mut)]
    pub user: UncheckedAccount<'info>,
    #[account(mut)]
    pub store_password_program: UncheckedAccount<'info>,
    pub system_program: Program<'info, System>,
}
```
