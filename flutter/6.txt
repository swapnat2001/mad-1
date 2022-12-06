import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';
import 'package:webfeed/webfeed.dart';
import 'package:http/http.dart' as http;
import 'package:url_launcher/url_launcher.dart';
void main() {
  runApp(const RSSDemo());
}

class RSSDemo extends StatelessWidget {
  const RSSDemo({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(title: "RSS Feed", home: RSSMainPicture());
  }
}

class RSSMainPicture extends StatefulWidget {
  const RSSMainPicture({Key? key}) : super(key: key);

  @override
  State<RSSMainPicture> createState() => _RSSMainPictureState();
}

class _RSSMainPictureState extends State<RSSMainPicture> {
  late Future<RssFeed> result;
  Future<RssFeed> giver() async {
    var response =
    await http.get(Uri.parse("https://www.espncricinfo.com/rss/content/story/feeds/0.xml"));
    var channel = RssFeed.parse(response.body);
    return channel;
  }

  @override
  void initState() {
    super.initState();
    result = giver();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text("News"),
        actions: [
          IconButton(onPressed: ()=>result=giver(), icon: const Icon(Icons.refresh_rounded)),
        ],
      ),
      body: FutureBuilder<RssFeed?>(
        future: result,
        builder: (context,snapshot){
          if(snapshot.hasError){
            if(kDebugMode){
              print("Error");
            }
            return Container();
          }
          else if(snapshot.connectionState==ConnectionState.waiting){
            return const Center(
              child: CircularProgressIndicator(),
            );
          }
          else if(snapshot.hasData){
            var feed=snapshot.data!;
            var items=feed.items;
            return ListView.builder(
              itemCount: items?.length,
              itemBuilder: (context,index){
                var item=items![index];
                return GestureDetector(
                  onTap: () async{
                    if (!await launchUrl(Uri.parse(item.link!))) {
                      throw 'Could not launch ${item.link}';
                    }
                  },
                  child: ListTile(
                    // leading: CachedNetworkImage(
                    //   imageUrl: mediaImage!,
                    //   progressIndicatorBuilder: (context, url, downloadProgress) =>
                    //       CircularProgressIndicator(value: downloadProgress.progress),
                    //   errorWidget: (context, url, error) => const Icon(Icons.error),
                    // ),
                    title: Text(item.title!),
                    subtitle: Text("${item.pubDate!}"),
                  ),
                );
              },
            );
          }
          return Container();
        },
      ),
    );
  }
}
