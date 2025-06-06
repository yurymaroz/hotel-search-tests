# QA Testing Plan for Flutter Hotel Booking App

## Manual QA Strategy

### Key User Flows

- Hotel Search with debounce and pagination
- Favorites management with Hive
- Navigation via auto_route
- State management with Bloc

## Detailed and Prioritized Test Scenarios

### 🔍 Hotel Search

| ID   | Priority | Scenario                              | Description / Steps                           | Expected Outcome                                             |
| ---- | -------- | ------------------------------------- | --------------------------------------------- | ------------------------------------------------------------ |
| HS01 | High     | Debounced search                      | Type quickly (`"Barc"`), pause                | One API call sent after pause                                |
| HS02 | High     | Paginated results loading             | Scroll to bottom of list with >10 items       | Next page of hotels fetched and appended                     |
| HS03 | Medium   | Search with no results                | Input gibberish or a location with no hotels  | Display "No hotels found" message                            |
| HS04 | Medium   | Search with invalid characters        | Input `!@#$%^&*()`                            | No crash, input is either ignored or results in "No results" |
| HS05 | Medium   | Search during slow network            | Throttle connection or simulate offline state | Retry indicator, no crash                                    |
| HS06 | Low      | Long query string                     | Input a 100+ character search term            | App truncates or handles without UI break                    |
| HS07 | High     | Cancelled search (replace input fast) | Input `"New"`, then quickly type `"York"`     | Only latest query (`"York"`) triggers results                |

### Favorites

| ID  | Priority | Scenario                         | Description / Steps                   | Expected Outcome                        |
| --- | -------- | -------------------------------- | ------------------------------------- | --------------------------------------- |
| F01 | High     | Add hotel to favorites           | Tap heart icon on hotel               | Icon fills, hotel saved in Hive         |
| F02 | High     | Remove hotel from favorites      | Tap heart icon again                  | Icon unfilled, hotel removed            |
| F03 | High     | Favorites persist on app restart | Favorite a hotel → close app → reopen | Favorites list remains unchanged        |
| F04 | Medium   | Navigate to favorites page       | Tap "Favorites" in menu or tab bar    | Favorites page loads                    |
| F05 | Medium   | Show empty state                 | No favorites exist                    | Display message: "No favorites yet"     |
| F06 | Edge     | Add same hotel multiple times    | Attempt to tap favorite repeatedly    | Hotel not duplicated                    |
| F07 | Edge     | Corrupted Hive box               | Tamper with local storage (test data) | App shows recovery or gracefully resets |

### Navigation

| ID  | Priority | Scenario                              | Description / Steps                             | Expected Outcome                 |
| --- | -------- | ------------------------------------- | ----------------------------------------------- | -------------------------------- |
| N01 | High     | Navigate between Dashboard/Favorites  | Use tab bar or menu                             | Correct pages shown, no crashes  |
| N02 | Medium   | Deep link navigation (if implemented) | Launch with URL or notification intent          | Correct page loaded              |
| N03 | Low      | Back button works correctly           | Navigate multiple pages, press Android/iOS back | Returns to correct previous page |

### UI Validations

| ID   | Priority | Scenario              | Description / Steps                         | Expected Outcome                          |
| ---- | -------- | --------------------- | ------------------------------------------- | ----------------------------------------- |
| UI01 | High     | Hotel card info       | Each card displays hotel name, image, price | All elements visible                      |
| UI02 | High     | Loading indicators    | While data fetching                         | Show shimmer or progress bar              |
| UI03 | Medium   | Responsive layout     | Rotate screen or test on tablet/phone       | Layout adjusts cleanly                    |
| UI04 | Medium   | Localization          | Switch device language                      | App uses localized strings (from `/i18n`) |
| UI05 | Edge     | Very long hotel names | Display hotel with long title               | Text wraps or truncates cleanly           |

### Bloc/Concurrency

| ID  | Priority | Scenario                                 | Description / Steps                              | Expected Outcome                               |
| --- | -------- | ---------------------------------------- | ------------------------------------------------ | ---------------------------------------------- |
| B01 | High     | Droppable prevents concurrent pagination | Scroll rapidly, triggering multiple page fetches | Only one fetch occurs, droppable strategy used |
| B02 | Medium   | BLoC state updates correctly             | Trigger add/remove from favorites in succession  | Correct latest state maintained                |
| B03 | Edge     | Bloc error handling                      | Simulate API failure or malformed response       | Emits failure state, UI displays error         |

---

## Widget & Integration Test Strategy

### Widget Test

```dart
testWidgets('Hotel card renders', (tester) async {
  await tester.pumpWidget(MaterialApp(home: HotelCard(...)));
  expect(find.text('Hotel Name'), findsOneWidget);
});
```

### Bloc Test

```dart
blocTest<FavoritesBloc, FavoritesState>(
  'emits updated state',
  build: () => favoritesBloc,
  act: (bloc) => bloc.add(AddHotelToFavorites(hotel)),
  expect: () => [FavoritesUpdated([...])],
);
```

### Integration Test Example

```dart
testWidgets('Search flow', (tester) async {
  app.main();
  await tester.pumpAndSettle();
  await tester.enterText(find.byType(TextField), 'Paris');
  await tester.pump(Duration(milliseconds: 500));
  expect(find.text('Paris'), findsWidgets);
});
```

---

## E2E with Maestro

### Maestro Flow Template (JSON)

```json
{
  "appId": "com.example.hotelapp",
  "name": "Search and Favorite Flow",
  "steps": [
    { "tapOn": "Search Hotels" },
    { "inputText": "Barcelona" },
    { "pressKey": "Enter" },
    { "waitForText": "Hotel Barcelona Center" },
    { "tapOn": "Hotel Barcelona Center" },
    { "tapOn": "Favorite" },
    { "tapOn": "Back" },
    { "tapOn": "Favorites" },
    { "assertVisible": "Hotel Barcelona Center" }
  ]
}
```

---

## Guidelines for Reviewing Test Results

- Use `flutter test --coverage` for coverage reports.
- Logs: `flutter run --verbose`
- Review integration and maestro output in CI logs or terminal.
