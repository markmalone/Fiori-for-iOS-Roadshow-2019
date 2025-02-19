# OData Query Examples

* [Getting a list of reports](#getlist)
* [Getting the list of expenses for a report](#getexpenses)
* [Creating a new expense](#createexpense)
* [Attach a receipt to an expense](#attachreceipt)
* [Updating an existing expense](#updateexpense)
* [Deleting an existing expense](#deleteexpense)
* [Submitting a report](#submitexpense)

This guide contains example code for you to get started with writing your own queries for the Travel Expense service.

The `dataService` in each example is expected to be the sample Travel Expense service. In an app generated by the Assistant, this is exposed as a property on the `ODataController` which can be accessed over the `AppDelegate` class. To access this more conveniently, the following lines can be placed at the top of your view controller classes:

```swift
guard let travelexpenseService = appDelegate.sessionManager.onboardingSession?.odataController.travelexpenseService else {
            AlertHelper.displayAlert(with: "OData service is not reachable, please onboard again.", error: nil, viewController: self)
            return
        }
self.travelexpenseService = travelexpenseService
```

The code in each of these examples can be used with any of the proxy classes. The data service provides `fetch` methods for each type, and any proxy class instance can be passed into the `createEntity`, `updateEntity`, and/or `deleteEntity` methods.

If you're interested on exploring the service through your browser you can do this as like this:

- ServiceDocument: <your-service-url>
- MetadataDocument: <your-service-url>/$metadata
- Reports: <your-service-url>/ReportSet
- Report with id 1: <your-service-url>/ReportSet(1)

You can do this with all the entities in the service.

<a name="getlist"/>

## Getting a list of reports

* Assume a property called `reports` of type `[Report]`.

```swift
// Order the results by the end date of the expense report, descending.
// Expand the query to also get the associated Report Status.
let query = DataQuery().orderBy(Report.end, .descending).expand(Report.reportStatus)

// Fetch all expense reports, using the query parameters defined above.
travelexpenseService?.fetchReportSet(matching: query) { [weak self] reports, error in
   if let error = error {
      NSLog("Error: %@", error!.localizedDescription)
      return
   }
   self?.reports = reports!
   self?.tableView.reloadData()
}
```

<a name="getexpenses">
   
## Getting the list of expenses for a report

When displaying the details for a selected report, you will likely want to display the list of associated expense items.

* Assume a property called `report` of type `Report`.
* Assume a property called `expenses` of type `[Expense]`.

```swift
// Filter the results by the selected report ID, automatically retrieving
// associated records for the expense type, attachments, currencies and ordering
// by the expense date, ascending.
let reportID = report.reportid!
let query = DataQuery().filter(Expense.reportID == reportID)
            .expand(Expense.expenseType, Expense.attachments, Expense.currency)
            .orderBy(Expense.date)
            
// Fetch all expense items, using the query parameters defined above.
travelexpenseService?.fetchExpenseSet(matching: query) { [weak self] expenses, error in
   // Make sure no errors occurred.
   if let error = error {
      NSLog("Error: %@", error!.localizedDescription)
      return
   }
   self?.expenses = expenses!
   self?.tableView.reloadData()
}
```

<a name="createexpense">
   
## Creating a new expense

* Assume a property called `report` of type `Report`.
* Assume a property called `newExpense` of type `Expense`.
* Assume there is a selected currency, payment type and expense type called `selectedCurrency`, `selectedPaymentType`, `selectedExpenseType`.
* Assume the properties of `newExpense` have been filled in from the UI.

```swift
let newExpense = Expense()
// Fill the expense properties

// Setting the corresponding IDs which are needed to make references to the navigation link properties
newExpense.reportID = reportID
newExpense.currencyID = selectedCurrency?.currencyID
newExpense.paymentTypeID = selectedPaymentType?.paymentTypeID
newExpense.expenseTypeID = selectedExpenseType?.expenseTypeID

travelexpenseService.createEntity(newExpense) {[weak self] error in
   if let error = error {
      NSLog("Error: %@", error.localizedDescription)
   }
   self?.tableView.reloadData()
}
```

<a name="attachreceipt">

## Attach a receipt to an expense

* Assume a property called `existingExpense` of type `Expense`.
* Assume the properties of `existingExpense` are already set and you want to attach an image.
* Assume the attachments are hold in an array of file URLs pointing to the taken pictures.

```swift
// Create the image data and add it to the newExpense object.
let imageData = try? Data(contentsOf: url)
let attachment = Attachment(withDefaults: false)
attachment.image = imageData

newExpense.attachments = [attachment]
            
travelexpenseService.createEntity(newExpense) {[weak self] error in
   if let error = error {
       AlertHelper.displayAlert(with: "Could not create Expense", error: error, viewController: self)
   }
   self?.tableView.reloadData()
}


```

<a name="updateexpense">
   
## Updating an existing expense

* Assume a property called `existingExpense` of type `Expense`.

```swift
dataService.updateEntity(existingExpense) {[weak self] error in
    // Make sure no errors occurred.
    if let error = error {
        NSLog("Error: %@", error.localizedDescription)
    }
    self?.tableView.reloadData()
}
```

<a name="deleteexpense">
   
## Deleting an existing expense

* Assume a property called `existingExpense` of type `Expense`.

```swift
dataService.deleteEntity(existingExpense) {[weak self] error in
    // Make sure no errors occurred.
    if let error = error {
        NSLog("Error: %@", error.localizedDescription)
    }
    self?.tableView.reloadData()
}
```

<a name="submitreport">

## Submitting a report

* Assume a property called `openReport` of type `Report`.
* Assume the id of the `In Review` report status is `2`.

Setting the `reportStatusID` to `2` will set the status of the report from `Open` to `In Review`. This is considered as submitting a report.

```swift
report.reportStatusID = 2

travelexpenseService.updateEntity(report) { [weak self] error in
   // Make sure no errors occurred.
    if let error = error {
        NSLog("Error: %@", error.localizedDescription)
    }
    self?.tableView.reloadData()
 }

```


