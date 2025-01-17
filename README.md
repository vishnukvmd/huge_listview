Huge ListView
=============

A performant `ListView` that can handle any number of items with ease. Unlike other infinite list approaches,
it doesn't just add new items to the list, growing to huge sizes in the end, but has a fixed size cache that
only keeps a handful of pages all the time, discarding old pages as new ones come in. The list asks for a pageful
of items at once, in an async function, expecting to receive a `Future<List<T>>` of your items.

Instead of a regular `ListView`, it uses a `ScrollablePositionedList` inside that makes it possible to scroll
to specific items rather than scroll positions. The items don't have to be of uniform height, their size differences
don't affect performance at all. This list implementation, however, doesn't play nice with the regular `Scrollbar`,
so we use our own `DraggableScrollbar` instead. This way we can support both a scrollbar and the usual position-based
scrolling as well.

The basic idea came from:

* https://stackoverflow.com/questions/60074466/pagination-infinite-scrolling-in-flutter-with-caching-and-realtime-invalidatio

The scrollbar is based on:

* https://pub.dev/packages/draggable_scrollbar

## Usage

``` dart
static const int PAGE_SIZE = 12;
final listKey = GlobalKey<HugeListViewState>();
final scroll = ItemScrollController();

HugeListView<MyDataItem>(
  /// Only needed if you expect to make use of its [setPosition] function.
  key: listKey,
  /// Only needed if you expect to make use of its [jumpTo] or [scrollTo] functions.
  controller: scroll,
  /// Size of the page. [HugeListView] only keeps a few pages of items in memory any time.
  pageSize: PAGE_SIZE,
  /// Total number of items in the list.
  totalCount: 999999,
  /// Index of an item to initially align within the viewport.
  startIndex: 0,
  /// Called to build items for the list with the specified [pageIndex].
  pageFuture: (page) => _loadPage(page, PAGE_SIZE),
  /// Called to build an individual item with the specified [index].
  itemBuilder: (context, index, MyDataItem entry) {
    return Text(entry.name);
  },
  /// Called to build the thumb. One of [DraggableScrollbarThumbs.RoundedRectThumb], [DraggableScrollbarThumbs.ArrowThumb],
  /// [DraggableScrollbarThumbs.SemicircleThumb] or build your own.
  thumbBuilder: DraggableScrollbarThumbs.SemicircleThumb,
  /// Background color of scroll thumb, defaults to white.
  thumbBackgroundColor: Colors.white,
  /// Drawing color of scroll thumb, defaults to gray.
  thumbDrawColor: Colors.grey,
  /// Height of scroll thumb, defaults to 48.
  thumbHeight: 48,
  /// Called to build a placeholder while the item is not yet availabe.
  placeholderBuilder: (context, index) => <some Widget>,
  /// Called to build a progress widget while the whole list is initialized.
  waitBuilder: (context) => <some Widget>,
  /// Called to build a widget when the list is empty.
  emptyResultBuilder: (context) => <some Widget>,
  /// Called to build a widget when there is an error.
  errorBuilder: (context, error) => <some Widget>,
  /// Event to call with the index of the topmost visible item in the viewport while scrolling.
  /// Can be used eg. to display the current letter of an alphabetically sorted list.
  firstShown: (index) {},
);
```

You have to pass a list of your items to `pageFuture`, for instance, given a list named `data`:

``` dart
Future<List<MyDataItem>> _loadPage(int page, int pageSize) async {
  int from = page * pageSize;
  int to = min(data.length, from + pageSize);
  return data.sublist(from, to);
}
```

The `waitBuilder` can be a simple centered `CircularProgressIndicator` but a nicer idea is
to provide a `placeholderBuilder` that is a mockup of the data to arrive. Many apps and sites
use gray horizontal bars instead of the actual text during loading. As an example,
here is a simple function that creates such a bar with randomly varying length:

``` dart
static const int PLACEHOLDER_SIZE = 14;

Widget buildPlaceholder() {
  double margin = Random().nextDouble() * 50;
  return Padding(
    padding: EdgeInsets.fromLTRB(3, 3, 3 + margin, 3),
    child: Container(
      height: PLACEHOLDER_SIZE,
      color: Colors.grey,
    ),
  );
}
```

You can pass it directly to `placeholderBuilder` and you can also use it to create
a whole page of text mockup bars that you can pass to `waitBuilder`:

``` dart
Widget buildWait() {
  return LayoutBuilder(
    builder: (_, constraints) {
      return ListView.builder(
        itemCount: (constraints.maxHeight / PLACEHOLDER_SIZE).ceil(),
        itemBuilder: (_, index) => buildPlaceholder(),
      );
    },
  );
}
```
