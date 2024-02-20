1. 创建Solana钱包和配置环境

```
solana-keygen new
solana config set --url https://api.testnet.solana.com # 设置为测试网
```

2. 安装Anchor框架和设置项目

```
npm install -g @project-serum/anchor
anchor init nft-project # 初始化项目
cd nft-project
```

3. lib.rs

```rust

#[derive(Accounts)]
pub struct NftMinting<'info> {
    /// CHECK: We're about to create this with Metaplex
    #[account(mut)]
    pub metadata: UncheckedAccount<'info>,
    /// CHECK: We're about to create this with Metaplex
    #[account(mut)]
    pub master_edition: UncheckedAccount<'info>,
    #[account(mut)]
    pub mint: Signer<'info>,
    /// CHECK: We're about to create this with Anchor
    #[account(mut)]
    pub token_account: UncheckedAccount<'info>,
    #[account(mut)]
    pub mint_authority: Signer<'info>,
    pub rent: Sysvar<'info, Rent>,
    pub system_program: Program<'info, System>,
    pub token_program: Program<'info, token::Token>,
    pub associated_token_program: Program<'info, associated_token::AssociatedToken>,
    /// CHECK: Metaplex will check this
    pub token_metadata_program: UncheckedAccount<'info>,
}
```

```rust

```