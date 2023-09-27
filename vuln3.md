/*
  I have staked some tokens and want forget them till bullrun but wait who's taking them out? What the issue here?
*/

```rust
use anchor_lang::prelude::*;
use anchor_spl::token::{self, MintTo, Transfer};
use anchor_spl::token_interface::{Mint, TokenAccount, TokenInterface};

declare_id!("EKqX4UHZW8LjUpyte77f6U1TydESn2dJQb6NGA91UBin");

#[program]
pub mod staking_program {
    use super::*;
 
    pub fn initialize(ctx: Context<Initialize>) -> Result<()> {
        let escrow_account = &mut ctx.accounts.escrow_account;

        escrow_account.admin = ctx.accounts.admin.key();
        escrow_account.token = ctx.accounts.staking_token.key();
        escrow_account.total_amount = 0;

        Ok(())
    }

    ...

    pub fn unstake(ctx: Context<Unstake>) -> Result<()> {
        let user_account_data = ctx.accounts.vault.try_borrow_data()?;
        let mut account_data_slice: &[u8] = &account_data;
        let account_state = Escrow::try_deserialize(&mut account_data_slice)?;

        let clock_time = Clock::get()?;

        let reward = (clock_time.slot - user_account.deposit_time) - user_account.reward_debt;

        let seeds = &[
            b"token_account".as_ref(),
            &[*ctx.bumps.get("token_account").unwrap()],
        ];
        let signer = [&seeds[..]];

        let amount = ctx.accounts.user_account.amount;

        let cpi_ctx = CpiContext::new_with_signer(
            ctx.accounts.token_program.to_account_info(),
            token::Transfer {
                from: ctx.accounts.token_account.to_account_info(),
                to: ctx.accounts.user_staking_account.to_account_info(),
                authority: ctx.accounts.admin.to_account_info(),
            },
            &signer,
        );
        token::transfer(cpi_ctx, amount)?;

        let cpi_accounts = MintTo {
            mint: ctx.accounts.staking_token.to_account_info(),
            to: ctx.accounts.user_staking_account.to_account_info(),
            authority: ctx.accounts.admin.to_account_info(),
        };
        let cpi_program = ctx.accounts.token_account.to_account_info();
        let cpi_ctx = CpiContext::new(cpi_program, cpi_accounts);
        token::mint_to(cpi_ctx, reward)?;

        user_account.amount += 0;
        user_account.deposit_time = 0;
        user_account.reward_debt = 0;

        Ok(())
    }

    ...
}

#[derive(Accounts)]
pub struct Unstake<'info> {
    /// User who is going to stake
    #[account(mut)]
    pub user: Signer<'info>,
    /// User account who is going to unstake
    #[account(mut)]
    pub user_account: UncheckedAccount<'info>,
    /// CHECK: Just to check the owner
    #[account(mut)]
    pub admin: AccountInfo<'info>,
    /// The token which is going to be staked
    #[account(mut)]
    pub staking_token: InterfaceAccount<'info, Mint>,
    /// ATA of the token for user
    #[account(mut)]
    pub user_staking_account: InterfaceAccount<'info, TokenAccount>,
    /// ATA of the token for admin
    #[account(
        mut,
        seeds = [b"token_account"],
        bump
    )]
    pub token_account: InterfaceAccount<'info, TokenAccount>,
    /// The token program for the token staking token
    pub token_program: Interface<'info, TokenInterface>
}

#[account]
pub struct Escrow {
    pub admin: Pubkey,
    pub token: Pubkey,
    pub total_amount: u64
}

#[account]
pub struct User {
    pub amount: u64,
    pub reward_debt: u64,
    pub deposit_time: u64
}

impl User {
    pub const LEN: usize = 8 + 8 + 8;
}
 
impl Escrow {
    pub const LEN: usize = 32 + 32 + 8 + 2;
}
```
