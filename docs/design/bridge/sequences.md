## Deposit

Bridge ERC 20 tokens from rootchain to childchain via deposit.

```mermaid
sequenceDiagram
	User->>Furtheon: deposit
	Furtheon->>RootERC20.sol: approve(RootERC20Predicate)
	Furtheon->>RootERC20Predicate.sol: deposit()
	RootERC20Predicate.sol->>RootERC20Predicate.sol: mapToken()
	RootERC20Predicate.sol->>StateSender.sol: syncState(MAP_TOKEN_SIG), recv=ChildERC20Predicate
	RootERC20Predicate.sol-->>Furtheon: TokenMapped Event
	StateSender.sol-->>Furtheon: StateSynced Event to map tokens on child predicate
	RootERC20Predicate.sol->>StateSender.sol: syncState(DEPOSIT_SIG), recv=ChildERC20Predicate
	StateSender.sol-->>Furtheon: StateSynced Event to deposit on child chain
	Furtheon->>User: ok
	Furtheon->>StateReceiver.sol:commit()
	StateReceiver.sol-->>Furtheon: NewCommitment Event
	Furtheon->>StateReceiver.sol:execute()
	StateReceiver.sol->>ChildERC20Predicate.sol:onStateReceive()
	ChildERC20Predicate.sol->>ChildERC20.sol: mint()
	StateReceiver.sol-->>Furtheon:StateSyncResult Event
```

## Withdraw

Bridge ERC 20 tokens from childchain to rootchain via withdrawal.

```mermaid
sequenceDiagram
	User->>Furtheon: withdraw
	Furtheon->>ChildERC20Predicate.sol: withdrawTo()
	ChildERC20Predicate.sol->>ChildERC20: burn()
	ChildERC20Predicate.sol->>L2StateSender.sol: syncState(WITHDRAW_SIG), recv=RootERC20Predicate
	Furtheon->>User: tx hash
	User->>Furtheon: get tx receipt
	Furtheon->>User: exit event id
	ChildERC20Predicate.sol-->>Furtheon: L2ERC20Withdraw Event
	L2StateSender.sol-->>Furtheon: StateSynced Event
	Furtheon->>Furtheon: Seal block
	Furtheon->>CheckpointManager.sol: submit()
```
## Exit

Finalize withdrawal of ERC 20 tokens from childchain to rootchain.

```mermaid
sequenceDiagram
	User->>Furtheon: exit, event id:X
	Furtheon->>Furtheon: bridge_generateExitProof()
	Furtheon->>CheckpointManager.sol: getCheckpointBlock()
	CheckpointManager.sol->>Furtheon: blockNum
	Furtheon->>Furtheon: getExitEventsForProof(epochNum, blockNum)
	Furtheon->>Furtheon: createExitTree(exitEvents)
	Furtheon->>Furtheon: generateProof()
	Furtheon->>ExitHelper.sol: exit()
	ExitHelper.sol->>CheckpointManager.sol: getEventMembershipByBlockNumber()
	ExitHelper.sol->>RootERC20Predicate.sol:onL2StateReceive()
	RootERC20Predicate.sol->>RootERC20: transfer()
	Furtheon->>User: ok
	RootERC20Predicate.sol-->>Furtheon: ERC20Withdraw Event
	ExitHelper.sol-->>Furtheon: ExitProcessed Event
```

