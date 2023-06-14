# Documentation of entities and their relationships

## Entity relationships of a contract

### Contract

A contract is a identifiable value bearing object to which a price can be associated. Contracts are grouped in two main categories:
- accounts
- (financial) instruments

#### Account

An account is an entity that describes a basket/container of financial instruments. Each account must be associated to a customer (typically the account holder and beneficial owner) and is usually enclosed in a portfolio. An account must by identified by a unique reference, the account number. It should also have an [IBAN](https://en.wikipedia.org/wiki/International_Bank_Account_Number) - International Bank Account Number - associated to it. There mainly two types of accounts:

- Customer Account
- Portfolio Account
- Cash Account
- Safekeeping Account

The cash account is as commonly understood a container for holdings of a currency. In this case the currency can be looked at as the financial instrument identified by its ISO3 currency code. In the case of a cash account the ladder holds only one financial instrument.
The safekeeping account is commonly used es a container for exchange traded instruments. The financial instrument is typically identified by an [ISIN](https://en.wikipedia.org/wiki/International_Securities_Identification_Number) - International Securities Identification Number. However there are many more instrument types that need to be considered, such as OTC products, like FX Forwards, currency & interest swaps, cryptocurrencies etc.

#### Financial Instrument

A financial instrument is an entity that desribes a monetary contract, which confers a right or claim against some counterparty in the form of a payment, equity ownership or dividends (stocks), debt (bonds, loans, deposit accounts), currency (forex), or derivatives (futures, forwards, options, and swaps). Financial instruments can be segmented by asset class, and as cash-based, securities, or derivatives.
An exchange traded financial instrument is typically identified by an [ISIN](https://en.wikipedia.org/wiki/International_Securities_Identification_Number) - International Securities Identification Number. Many other instrument types such as OTC products, like FX Forwards, currency & interest swaps, cryptocurrencies etc. have proprietary ways of identification.
In the context of OpenWealth the identification of the financial instrument is mandatory. In addition all properties (such as contract size, price factor, denomination, derivative figures etc.) required for a correct valuation of the instrument should be added to the properties. There are many classification schemes available for the classification of an instrument. OpenWealth proposes to use ISO 10962 as the standard classification of an instrument

```mermaid
erDiagram
    Identifier }|--|| Contract : ContractId
    Entry }|--|| Contract : AccountId
    Entry }|--|| Contract : InstrumentId
    Balance }|--|| Contract : AccountId
    Balance }|--|| Contract : InstrumentId
    Figure }|--|| Contract : ContractId
    Contract }|--|| Contract : AccountId
    Contract }|--|| Contract : UnderlyingContractId
    Contract {
        uuid Id PK
        string Name "Name of the contract - instrument or account designation"
        string Description "Description of the contract"
        enum Type "Type of the contract - cash account, security etc."
        string Currency "Denomination currency of the contract"
        decimal Size "Contract size, for example for options & futures"
        decimal(null) HasFactor "Indication of additional factor for valuation"
        decimal(null) Factor "Additional factor that needs to be applied for valuation"
        date(null) ExpiryDate "Date when the contract expires"
        uuid(null) AccountId FK "Parent account relation of the contract"
        uuid(null) UnderlyingContractId FK "Underlying contract of an instrument if applicable"
        enum(null) Custodian "Custodian that holds/issued contract/account"
    }
    Identifier {
        uuid Id PK
        enum Type "Type of the identifier"
        string Val "Id/Value of the identifier"
        uuid ContractId "Contract which the identifier identifies"
    }
```

## Entity relationships of a transaction

### Transaction

A transaction is an entity that describes an agreement of trade of financial wealth. Properties are the transaction type, dates related to the agreement, related account and customer, etc. Examples of transactions simple transfer of cash, a purchase of exchange-traded security, a confirmation of revenue or right distribution of a holding, a forward foreign exchange transaction, etc. A transaction encloses one or more movements of financial instruments in a given account.

### Entry

An entry, aka movement, is an entity that describes a change in units of a particular financial instrument in a given account. Typically a prize is associated with the movement, for example, the purchase price of a security. A transaction encloses one or more movements of one or more financial instruments. A purchase of a stock for example can consist of the purches units of the particular stock and a number of movements on the associated cash account, such as gross amount (units * price), brokerage fee, stamp and/or other local taxes.

```mermaid
erDiagram
    Transaction ||--|{ Entry : TransactionId
    Transaction }|--o| Account : AccountId
    Transaction }|--o| Instrument : TriggeringInstrumentId
    Transaction {
        uuid Id PK
        enum Type "Transaction type"
        enum Status "Status of the transaction"
        bool IsReversal "Cancellation/reversal indicator"
        date TransactionDate "Date of the transaction - i.e. trade date"
        uuid AccountId "The account where the transaction occurred in, can be the customer"
        uuid(null) TriggeringContractId "The contract - account or instrument - that caused the transaction"
        string(null) ExtRef "The banks identification of the transaction"
        string(null) ExtLinkRef "The banks reference to a linked transaction"
        string(null) Description "Description of the transaction - usually as delivered by the bank"
    }
    Entry }|--|| Account : AccountId
    Entry }|--|| Instrument : InstrumentId
    Entry {
        uuid Id PK
        uuid TransactionId FK "Transaction where the movement ocuurred in"
        enum Type "Entry type"
        date EntryDate "Date when the movement was is booked - typically the bookdate by the bank"
        date(null) ValueDate "Value date for cash movements, relevant for interest"
        uuid AccountId FK "Account in which the movement occurred"
        uuid InstrumentId FK "Instrument subject to the movemnt"
        decimal Units "Amount of units that changed"
        decimal(null) Price "Market or indication price of the instrument at the time of movement"
        enum(null) PriceType "Price type, i.e. PerUnit or Percent"
        decimal(null) FxRate "Foreign exchange rate that applied on the movement"
        string(null) FxPair "Source and target currencies of the FX concatenated"
        decimal(null) CostPrice "Cost price of the lot if applicable, i.e. relvant for deliveries or specific corporate actions"
        decimal(null) CostFxRate "Cost FX that was applied, should be to reference currency of portfolio"
        string(null) CostFxPair "Source and target currencies of the Cost FX concatenated"
        int(null) MessageId "Message from the entry originated"
        int(null) MessageIndex "Index in the message from where the movement originated"
    }
    Account {
        uuid Id PK
        uuid(null) AccountId FK "Parent Account if applicable"
        enum Type "Account type - cash account, safekeeping account etc"
        string Name
        string Currency
        string Nr "Account Nr"
        string IBAN "IBAN of the account"
    }
    Instrument {
        uuid Id PK
        enum Type "Instrument type - security, forward contract, money market contract etc."
        string Name
        string Currency
        string ISIN "IBAN of the account"
        decimal(null) Size "Contract size of the financial instrument"
        date(null) ExpiryDate "Expiry Date of the financial instrument"
    }
```

## Entity relationships of a balance

### Balance

A balance is an entity that describes the amount or units held of a particular finacial instrument in a given account at a particular date in time.
Normally valuation properties such as price, accrued and fx are associated to the balance

```mermaid
erDiagram
    Balance }|--|| Account : AccountId
    Balance }|--|| Instrument : InstrumentId
    Balance {
        uuid Id PK
        enum Status "Status of the balance"
        date BalanceDate "Date of the balance"
        uuid AccountId FK "Account of the balance"
        uuid InstrumentId FK "Instrument to the balance"
        decimal Units "Amount of units of the balance"
        decimal(null) Price "Market or indication price of the balance"
        string(null) PriceCurrency "Currency of the price"
        enum(null) PriceType "Price type, i.e. PerUnit or Percent"
        decimal(null) Accrued "Accrued interest of the balance"
        string(null) AccruedCurrency "Currency of the accrued interest"
        decimal(null) FxRate "Foreign exchange rate that applied on the movement"
        string(null) FxPair "Source and target currencies of the FX concatenated"
        decimal(null) CostPrice "Cost price of the lot if applicable, i.e. relvant for deliveries or specific corporate actions"
        string(null) CostCurrency "Currency of the cost price"
        decimal(null) CostFX "Cost FX that was applied, should be to reference currency of portfolio"
        string(null) CostFXCurrencyPair "Source and target currencies of the Cost FX concatenated"
        enum(null) Source "Source, typically custodian, of the balance"
        uuid(null) MessageId "Message from the balance originated"
        int(null) MessageIndex "Index in the message from where the balance originated"
    }
```

## Entity relationships of a figure

### Figure

A figure is an entity the describes a numeric characteristic of a contrct at a given date, for example a price quote, accrued interest or a rate usch as foreign exchange, fx forward etc.

```mermaid
erDiagram
    Figure }|--|| Contract : ContractId
    Figure {
        int Id PK
        enum Status "Status of the figure"
        enum Type "Type of the figure"
        date FigureDate "Date of the figure"
        int ContractId FK "Contract of the figure"
        decimal Val "Figure value"
        string(null) Currency "Currency of the figure"
        enum(null) CalcMethod "Calculation method of the figure, i.e. interest day count"
        enum(null) Source "Source, typically custodian, of the figure"
        int(null) MessageId "Message from the figure originated"
        int(null) MessageIndex "Index in the message from where the figure originated"
    }
```

## Entity relationships of a cash (net cash movement)

### Cash
Cash is an entity provided by the custodian bank containing the information of a net movement on a cash account

```mermaid
erDiagram
    Cash }|--|| Account : AccountId
    Cash {
        int Id PK
        enum Status "Status of the cash"
        date BookDate "Book date of the cash"
        date ValueDate "Value date of the cash"
        int AccountId FK "Account of the cash"
        enum Type "Transaction type"
        bool IsReversal "Cancellation/reversal indicator"
        decimal Units "Units of change"
        int(null) MessageId "Message from the cash originated"
        int(null) MessageIndex "Index in the message from where the cash originated"
    }
```