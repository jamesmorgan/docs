# Client Method Reference

## Core Channel Management Methods
```javascript
transfer: (TransferParams) =>  Promise<ChannelState>
deposit: (DepositParams) => Promise<ChannelState>
exchange: (ExchangeParams) => Promise<ChannelState>
on: (event: ConnextEvent, (cb: any => any) => void) => void
recipientNeedsCollateral: (string, PartialPayment Payment) => Promise<string|null>
redeem: (string) => Promise<string>
withdraw: (WithdrawParams) => Promise<ChannelState>
installApp: (InstallAppParams) => Promise<ChannelState>
uninstallApp: (UninstallAppParams) => Promise<ChannelState>
takeAction: (takeActionParams) => Promise<AppState>
getHistory: () => Promise<AppState[]>
getHistoryById: (id) => Promise<AppState>
getProfileConfig: () => Promise<PaymentProfile>
startProfileSession: (string) => Promise<void>
getChannelState: () => Promise<ChannelState>
getAppState: (string) => Promise<AppState>
getApps: () => Promise<string[]>
getFreeBalance: () => //TODO
```

## Low Level Channel API (mapped to CF node)
```javascript
createChannel: () => Promise<string>
proposeInstall: (LowLevelProposeInstallParams) => Promise<string>
install: (LowLevelInstallParams) => Promise<AppInstance>
rejectInstall: (LowLevelRejectInstallParams) => void
proposeInstallVirtual: (LowLevelProposeInstallVirtualParams) => Promise<string>
installVirtual: (LowLevelInstallVirtualParams) => Promise<AppInstance>
proposeState: (LowLevelProposeStateParams) => //TODO (CF)
updateState: (LowLevelUpdateStateParams) => //TODO (CF)
rejectState: (LowLevelRejectStateParams) => //TODO (CF)
getProposedAppInstance: () => Promise<AppInstance[]>
```
