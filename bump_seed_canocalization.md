For generating program address solana uses 256-bit pre-image resistant hash function using collection of seeds and a program id as input,
as it's output can't be predicted in advance and can't be controlled in any manner there is 50% chance of newly generated address will lie on ed25519 curve.
In such case where generated address is on curve, which is prohibited by solana. Solana will use a different set of seeds or a seed bump to find a valid address (address off the curve).

Excerpt from official Solana Docs

>Program addresses are deterministically derived from a collection of seeds and a program id using a 256-bit pre-image resistant hash function. 
>Program address must not lie on the ed25519 curve to ensure there is no associated private key. During generation, an error will be returned if the address is found to lie on the curve. 
>There is about a 50/50 chance of this happening for a given collection of seeds and program id. If this occurs a different set of seeds or a seed bump (additional 8 bit seed) can be used to find a valid program address off the curve.

The thing about PDA's is that they are found rather than created outside the ed2559 curve. The highest value for bump value is `bump = 255` and it goes decreasing `bump = 254`, `bump = 253` to find the derived address.
It is technically possible that no bump is found within 256 tries but this probability is negligible.

The first bump that results in a PDA is commonly called the "canonical bump". Other bumps may also result in a PDA but it's recommended to only use the canonical bump to avoid confusion.
```rust
#[derive(Accounts)]
pub struct CreateEscrow<'info> {
	#[account(
    	seeds = [ESCROW_PDA_SEED.as_bytes()],
	    bump = escrow.bump
	)]
	escrow: Account<'info, Escrow>,
}
```

- `create_program_address()` function in solana does creates a program address but without valid bump.
```rust
let (expected_pda, bump_seed) = Pubkey::find_program_address(&[b"vault"], &program_id);
let actual_pda = Pubkey::create_program_address(&[b"vault", &[bump_seed]], &program_id)?;
assert_eq!(expected_pda, actual_pda);
```

```rust
use anchor_lang::prelude::*;

declare_id!("Fg6PaFpoGXkYsidMpWTK6W2BeZ7FEfcYkg476zPFsLnS");

#[program]
pub mod bump_seed_canonicalization {
    use super::*;

    pub fn set_data(ctx: Context<BumpSeed>, key: u64, bump: u8, new_value: u64, project_name: String, link: String) -> Result<()> {
        let address =
            Pubkey::create_program_address(&[key.to_le_bytes().as_ref(), &[bump]], ctx.program_id).unwrap();
        if address != ctx.accounts.bump_val_account.key() {
            return Err(ProgramError::InvalidArgument.into());
        }

        ctx.accounts.bump_val_account.data = new_value;
        ctx.accounts.bump_val_account.name = name;
        ctx.accounts.bump_val_account.link = link;

        Ok(())
    }
}

#[derive(Accounts)]
pub struct BumpSeed<'info> {
    bump_val_account: Account<'info, Data>,
}

#[account]
pub struct ProjectData {
    name: String,
    link: String,
    data: u64,
}
```

- `find_program_address()` function helps in finding valid address with help of bumps and seeds.
```rust
let (address, bump_seed) =
                Pubkey::find_program_address(&[b"Lil'", b"Bits"], &program_id);
```

```rust
pub fn create_escrow_account(ctx<CreateEscrow>, new_value: u64) -> Result<()> {
  let (escrow_address, escrow_bump_seed) =
                Pubkey::find_program_address(&[b"escrow"], ctx.program_id);

    if escrow_address != ctx.accounts.escrow.key() {
        return Err(ProgramError::InvalidArgument.into());
    }

    ctx.accounts.escrow.data = new_value;
    Ok(())
}

#[derive(Accounts)]
pub struct CreateEscrow<'info> {
  escrow: <'info, Escrow>,
  system_program: <'info, System>,
}

#[account]
pub struct Escrow {
  data: u64,
}
```

- Using anchor's bump and seed constraint

```rust
pub mod bump_seed_canonicalization {
    use super::*;

    pub fn set_data(ctx: Context<CreateEscrow>, new_value: u64) -> Result<()> {
        let escrow = &mut ctx.accounts.escrow;
        escrow.data = new_value;

        Ok(())
    }
}

#[derive(Accounts)]
pub struct CreateEscrow<'info> {
  #[account(
      seeds = [b"escrow"],
      bump,
  )]
  escrow: <'info, Escrow>,
  system_program: <'info, System>,
}

#[account]
pub struct Escrow {
  data: u64,
}
```

