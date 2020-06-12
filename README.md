# Flutter Search Bar

## Algorithm

``` dart
// This source code is a part of Project Violet.
// Copyright (C) 2020. violet-team. Licensed under the MIT Licence.

// Implementation of distance functions that satisfy trigonometry.

import 'dart:math';

class Distance {
  static int hammingDistance<T extends Comparable<T>>(List<T> l1, List<T> l2) {
    if (l1.length != l2.length) return -1;

    int result = 0;
    for (int i = 0; i < l1.length; i++) if (l1[i] != l2[i]) result++;

    return result;
  }

  static int levenshteinDistance<T extends Comparable<T>>(List<T> l1, List<T> l2) {
    int x = l1.length;
    int y = l2.length;
    int i, j;

    if (x == 0) return x;
    if (y == 0) return y;
    var v0 = List<int>((y + 1) << 1);

    for (i = 0; i < y + 1; i++) v0[i] = i;
    for (i = 0; i < x; i++) {
      v0[y + 1] = i + 1;
      for (j = 0; j < y; j++)
        v0[y + j + 2] = min(min(v0[y + j + 1], v0[j + 1]) + 1,
            v0[j] + ((l1[i] == l2[j]) ? 0 : 1));
      for (j = 0; j < y + 1; j++) v0[j] = v0[y + j + 1];
    }

    return v0[y + y + 1];
  }
}
```

## Text Clustering

## Elastic Search

## UI

``` dart
class SearchBar extends StatefulWidget {
  final FlareControls heroController;
  final FlutterActorArtboard artboard;
  const SearchBar({Key key, this.artboard, this.heroController})
      : super(key: key);

  @override
  _SearchBarState createState() => _SearchBarState();
}

class _SearchBarState extends State<SearchBar>
    with SingleTickerProviderStateMixin {
  AnimationController controller;
  List<Tuple3<String, String, int>> _searchLists =
      List<Tuple3<String, String, int>>();

  TextEditingController _searchController = TextEditingController();
  int _insertPos, _insertLength;
  String _searchText;
  bool _nothing = false;
  bool _onChip = false;
  bool _tagTranslation = false;
  bool _showCount = true;
  int _searchResultMaximum = 60;

  @override
  void initState() {
    super.initState();
    controller = AnimationController(
      vsync: this,
      duration: Duration(milliseconds: 400),
      reverseDuration: Duration(milliseconds: 400),
    );
  }

  @override
  Widget build(BuildContext context) {
    final double statusBarHeight = MediaQuery.of(context).padding.top;
    controller.forward();

    if (_searchLists.length == 0 && !_nothing) 
      _searchLists.add(Tuple3<String, String, int>('prefix', 'tag', 0));
      _searchLists.add(Tuple3<String, String, int>('prefix', 'lang', 0));
      _searchLists.add(Tuple3<String, String, int>('prefix', 'series', 0));
      _searchLists.add(Tuple3<String, String, int>('prefix', 'author', 0));
      _searchLists.add(Tuple3<String, String, int>('prefix', 'type', 0));
      _searchLists.add(Tuple3<String, String, int>('prefix', 'class', 0));
      _searchLists.add(Tuple3<String, String, int>('prefix', 'recent', 0));
    }

    return Container(
        color: Colors.white,
        padding: EdgeInsets.fromLTRB(2, statusBarHeight + 2, 0, 0),
        child: Stack(children: <Widget>[
          Hero(
            tag: "searchbar",
            child: Card(
              elevation: 100,
              shape: RoundedRectangleBorder(
                  borderRadius: BorderRadius.circular(4)),
              child: Material(
                child: Container(
                  child: Column(
                    children: <Widget>[
                      Material(
                        child: ListTile(
                          title: TextFormField(
                            cursorColor: Colors.black,
                            onChanged: (String str) async {
                              await searchProcess(
                                  str, _searchController.selection);
                            },
                            controller: _searchController,
                            decoration: new InputDecoration(
                              border: InputBorder.none,
                              focusedBorder: InputBorder.none,
                              enabledBorder: InputBorder.none,
                              errorBorder: InputBorder.none,
                              disabledBorder: InputBorder.none,
                              suffixIcon: IconButton(
                                onPressed: () async {
                                  _searchController.clear();
                                  _searchController.selection = TextSelection(
                                      baseOffset: 0, extentOffset: 0);
                                  await searchProcess(
                                      '', _searchController.selection);
                                },
                                icon: Icon(Icons.clear),
                              ),
                              contentPadding: EdgeInsets.only(
                                  left: 15, bottom: 11, top: 11, right: 15),
                              hintText: '검색',
                            ),
                          ),
                          leading: SizedBox(
                            width: 25,
                            height: 25,
                            child: RawMaterialButton(
                                onPressed: () {
                                  Navigator.pop(context);
                                },
                                shape: CircleBorder(),
                                child: FlareArtboard(widget.artboard,
                                    controller: widget.heroController)),
                          ),
                        ),
                      ),
                      Padding(
                        padding: EdgeInsets.symmetric(horizontal: 10.0),
                        child: Container(
                          height: 1.0,
                          color: Colors.black12,
                        ),
                      ),
                      SizedBox(
                        height: 40,
                        child: Padding(
                          padding: EdgeInsets.fromLTRB(8, 2, 8, 2),
                          child: ButtonTheme(
                            minWidth: double.infinity,
                            height: 30,
                            child: RaisedButton(
                              color: Colors.purple,
                              textColor: Colors.white,
                              child: Text('검색'),
                              onPressed: () async {
                                final query = TokenManager.translate2query(
                                    _searchController.text);
                                final result =
                                    QueryManager.queryPagination(query);
                                Navigator.pop(
                                    context,
                                    Tuple2<QueryManager, String>(
                                        result, _searchController.text));
                              },
                            ),
                          ),
                        ),
                      ),
                      Padding(
                        padding: EdgeInsets.symmetric(horizontal: 10.0),
                        child: Container(
                          height: 1.0,
                          color: Colors.black12,
                        ),
                      ),
                      Expanded(
                        child: _searchLists.length == 0 || _nothing
                            ? Center(
                                child: Text(_nothing
                                    ? '검색 결과가 없습니다 :('
                                    : "검색어를 입력해 주세요!"))
                            : Padding(
                                padding: EdgeInsets.symmetric(horizontal: 8),
                                child: FadingEdgeScrollView
                                    .fromSingleChildScrollView(
                                  child: SingleChildScrollView(
                                    controller: ScrollController(),
                                    child: Wrap(
                                      spacing: 4.0,
                                      runSpacing: -10.0,
                                      children: _searchLists
                                          .map((item) => chip(item))
                                          .toList(),
                                    ),
                                  ),
                                  gradientFractionOnEnd: 0.1,
                                  gradientFractionOnStart: 0.1,
                                ),
                              ),
                      ),
                      Padding(
                        padding: EdgeInsets.symmetric(horizontal: 10.0),
                        child: Container(
                          height: 1.0,
                          color: Colors.black12,
                        ),
                      ),
                      Expanded(
                        child: Padding(
                          padding: EdgeInsets.fromLTRB(10, 4, 10, 4),
                          child: LayoutBuilder(builder: (BuildContext context,
                              BoxConstraints constraints) {
                            return SingleChildScrollView(
                              controller: ScrollController(),
                              child: ConstrainedBox(
                                constraints: constraints.copyWith(
                                  minHeight: constraints.maxHeight,
                                  maxHeight: double.infinity,
                                ),
                                child: IntrinsicHeight(
                                  child: Column(
                                    mainAxisAlignment: MainAxisAlignment.start,
                                    crossAxisAlignment:
                                        CrossAxisAlignment.stretch,
                                    children: <Widget>[
                                      ListTile(
                                        leading: Icon(Icons.translate,
                                            color: Colors.purple),
                                        title: Text("태그 한글화"),
                                        trailing: Switch(
                                          value: _tagTranslation,
                                          onChanged: (value) {
                                            setState(() {
                                              _tagTranslation = value;
                                            });
                                          },
                                          activeTrackColor: Colors.purple,
                                          activeColor: Colors.purpleAccent,
                                        ),
                                      ),
                                      Container(
                                        margin: const EdgeInsets.symmetric(
                                          horizontal: 8.0,
                                        ),
                                        width: double.infinity,
                                        height: 1.0,
                                        color: Colors.grey.shade400,
                                      ),
                                      ListTile(
                                        leading: Icon(MdiIcons.counter,
                                            color: Colors.purple),
                                        title: Text("카운트 표시"),
                                        trailing: Switch(
                                          value: _showCount,
                                          onChanged: (value) {
                                            setState(() {
                                              _showCount = value;
                                            });
                                          },
                                          activeTrackColor: Colors.purple,
                                          activeColor: Colors.purpleAccent,
                                        ),
                                      ),
                                      Expanded(
                                        child: Align(
                                          alignment: Alignment.bottomLeft,
                                          child: Container(
                                            margin: const EdgeInsets.symmetric(
                                              horizontal: 2,
                                              vertical: 8,
                                            ),
                                            width: double.infinity,
                                            height: 60,
                                            child: RaisedButton(
                                              child: Icon(MdiIcons.autoFix),
                                              onPressed: () {
                                                magicProcess();
                                              },
                                            ),
                                          ),
                                        ),
                                      ),
                                    ],
                                  ),
                                ),
                              ),
                            );
                          }),
                        ),
                      ),
                    ],
                  ),
                ),
              ),
            ),
          ),
        ]));
  }

  Future<void> searchProcess(String target, TextSelection selection) async {
    _nothing = false;
    _onChip = false;
    if (target.trim() == '') {
      setState(() {
        _searchLists.clear();
      });
      return;
    }

    int pos = selection.base.offset - 1;
    for (; pos > 0; pos--)
      if (target[pos] == ' ') {
        pos++;
        break;
      }

    var last = target.indexOf(' ', pos);
    var token =
        target.substring(pos, last == -1 ? target.length : last + 1).trim();

    if (pos != target.length && target[pos] == '-') {
      token = token.substring(1);
      pos++;
    }
    if (token == '') {
      setState(() {
        _searchLists.clear();
      });
      return;
    }

    _insertPos = pos;
    _insertLength = token.length;
    _searchText = target;
    final result = (await TokenManager.queryAutoComplete(token))
        .take(_searchResultMaximum)
        .toList();
    if (result.length == 0) _nothing = true;
    setState(() {
      _searchLists = result;
    });
  }

  void magicProcess() {
    var text = _searchController.text;
    var selection = _searchController.selection;

    if (_nothing) {
      if (text == null || text.trim() == '')
        return;

      // DateTime()
    }


  }

  // Create tag-chip
  // group, name, counts
  Widget chip(Tuple3<String, String, int> info) {
    var tagRaw = info.item2;
    var count = '';
    var color = Colors.grey;

    if (_tagTranslation) // Korean
      tagRaw =
          TokenManager.mapSeries2Kor(TokenManager.mapTag2Kor(info.item2));

    if (info.item3 > 0 && _showCount) count = ' (${info.item3})';

    if (info.item1 == 'female')
      color = Colors.pink;
    else if (info.item1 == 'male')
      color = Colors.blue;
    else if (info.item1 == 'prefix') color = Colors.orange;

    var fc = RawChip(
      labelPadding: EdgeInsets.all(0.0),
      avatar: CircleAvatar(
        backgroundColor: Colors.grey.shade600,
        child: Text(info.item1[0].toUpperCase()),
      ),
      label: Text(
        ' ' + tagRaw + count,
        style: TextStyle(
          color: Colors.white,
        ),
      ),
      backgroundColor: color,
      elevation: 6.0,
      shadowColor: Colors.grey[60],
      padding: EdgeInsets.all(6.0),
      onPressed: () async {
        if (info.item1 != 'prefix') {
          var insert = info.item2.replaceAll(' ', '_');
          if (info.item1 != 'female' && info.item1 != 'male')
            insert = info.item1 + ':' + insert;

          _searchController.text = _searchText.substring(0, _insertPos) +
              insert +
              _searchText.substring(
                  _insertPos + _insertLength, _searchText.length);
          _searchController.selection = TextSelection(
            baseOffset: _insertPos + insert.length,
            extentOffset: _insertPos + insert.length,
          );
        } else {
          var offset = _searchController.selection.baseOffset;
          if (offset != -1) {
            _searchController.text = _searchController.text
                    .substring(0, _searchController.selection.base.offset) +
                info.item2 +
                ': ' +
                _searchController.text
                    .substring(_searchController.selection.base.offset);
            _searchController.selection = TextSelection(
              baseOffset: offset + info.item2.length + 1,
              extentOffset: offset + info.item2.length + 1,
            );
          } else {
            _searchController.text = info.item2 + ': ';
            _searchController.selection = TextSelection(
              baseOffset: info.item2.length + 1,
              extentOffset: info.item2.length + 1,
            );
          }
          _onChip = true;
          await searchProcess(
              _searchController.text, _searchController.selection);
        }
      },
    );
    return fc;
  }
}
```
