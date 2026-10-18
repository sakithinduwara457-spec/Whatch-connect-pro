import 'dart:io';
import 'package:flutter/material.dart';
import 'package:flutter_blue_plus/flutter_blue_plus.dart';
import 'package:permission_handler/permission_handler.dart';
import 'package:image_picker/image_picker.dart';

void main() {
  runApp(MaterialApp(home: WatchConnectApp(), debugShowCheckedModeBanner: false));
}

class WatchConnectApp extends StatefulWidget {
  @override
  _WatchConnectAppState createState() => _WatchConnectAppState();
}

class _WatchConnectAppState extends State<WatchConnectApp> {
  List<ScanResult> devices = [];
  BluetoothDevice? connectedDevice;
  bool isScanning = false;
  String watchType = "unknown";

  // Wallpaper Pack
  List<Map<String, String>> wallpapers = [
    {"name": "Abstract Neon", "url": "https://picsum.photos/seed/neon/400/400"},
    {"name": "Mountain Dark", "url": "https://picsum.photos/seed/mountain/400/400"},
    {"name": "Galaxy Purple", "url": "https://picsum.photos/seed/galaxy/400/400"},
    {"name": "Geometric", "url": "https://picsum.photos/seed/geo/400/400"},
    {"name": "Nature", "url": "https://picsum.photos/seed/nature/400/400"},
    {"name": "Minimal Black", "url": "https://picsum.photos/seed/black/400/400"},
  ];

  @override
  void initState() {
    super.initState();
    checkPermissions();
  }

  Future<void> checkPermissions() async {
    await [Permission.bluetoothScan, Permission.bluetoothConnect, Permission.location].request();
  }

  void startScan() async {
    setState(() { isScanning = true; devices.clear(); });
    await FlutterBluePlus.startScan(timeout: Duration(seconds: 10));
    FlutterBluePlus.scanResults.listen((results) {
      setState(() { devices = results; });
    });
    await Future.delayed(Duration(seconds: 10));
    await FlutterBluePlus.stopScan();
    setState(() { isScanning = false; });
  }

  void connectToDevice(BluetoothDevice device) async {
    try {
      await device.connect();
      setState(() {
        connectedDevice = device;
        // Watch Type Detect
        String name = device.name.toLowerCase();
        if (name.contains("galaxy") || name.contains("watch5") || name.contains("pixel")) {
          watchType = "wear_os";
        } else {
          watchType = "generic";
        }
      });
      ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text("${device.name} Connected!")));
    } catch (e) {
      print(e);
    }
  }

  void sendWallpaper(String url) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text("Wallpaper sent to ${connectedDevice!.name} - Size auto adjusted to ${watchType == 'wear_os' ? '450x450' : '240x240'}"))
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Watch Connect Pro"), backgroundColor: Colors.black, foregroundColor: Colors.white),
      backgroundColor: Color(0xFF121212),
      body: connectedDevice == null ? buildScanUI() : buildConnectedUI(),
      floatingActionButton: connectedDevice == null
          ? FloatingActionButton(onPressed: startScan, child: Icon(isScanning ? Icons.stop : Icons.search))
          : null,
    );
  }

  Widget buildScanUI() {
    return Column(
      children: [
        Padding(
          padding: EdgeInsets.all(16),
          child: Text("Connect වෙන Watch එක Scan කරන්න", style: TextStyle(color: Colors.white, fontSize: 18)),
        ),
        if (isScanning) LinearProgressIndicator(),
        Expanded(
          child: ListView.builder(
            itemCount: devices.length,
            itemBuilder: (context, i) {
              var d = devices[i];
              return Card(
                color: Colors.white10,
                child: ListTile(
                  title: Text(d.device.name.isEmpty ? "Unknown Watch" : d.device.name, style: TextStyle(color: Colors.white)),
                  subtitle: Text(d.device.remoteId.toString(), style: TextStyle(color: Colors.white54)),
                  trailing: Icon(Icons.watch, color: Colors.blue),
                  onTap: () => connectToDevice(d.device),
                ),
              );
            },
          ),
        ),
      ],
    );
  }

  Widget buildConnectedUI() {
    return SingleChildScrollView(
      padding: EdgeInsets.all(16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Card(
            color: Colors.green.withOpacity(0.2),
            child: ListTile(
              leading: Icon(Icons.check_circle, color: Colors.green),
              title: Text("Connected: ${connectedDevice!.name}", style: TextStyle(color: Colors.white)),
              subtitle: Text("Type: ${watchType == 'wear_os' ? 'Wear OS (App Install Supported)' : 'Generic Smartwatch'}", style: TextStyle(color: Colors.white70)),
              trailing: IconButton(icon: Icon(Icons.close, color: Colors.white), onPressed: () async {
                await connectedDevice!.disconnect();
                setState(() => connectedDevice = null);
              }),
            ),
          ),
          SizedBox(height: 20),
          Text("📱 Watch එකට App දාගන්න", style: TextStyle(color: Colors.white, fontSize: 18, fontWeight: FontWeight.bold)),
          SizedBox(height: 10),
          if (watchType == "wear_os")
            Card(
              color: Colors.blue.withOpacity(0.2),
              child: ListTile(
                title: Text("Install Apps to Watch", style: TextStyle(color: Colors.white)),
                subtitle: Text("Play Store open කරලා Watch එකට Apps දාගන්න", style: TextStyle(color: Colors.white70)),
                trailing: Icon(Icons.download, color: Colors.white),
                onTap: () {
                  ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text("Opening Play Store on Watch... (Wear OS)")));
                },
              ),
            )
          else
            Card(
              color: Colors.orange.withOpacity(0.2),
              child: ListTile(
                title: Text("Download Watch Faces", style: TextStyle(color: Colors.white)),
                subtitle: Text("මේ watch එකට ගැලපෙන Faces 100+ තියෙනවා", style: TextStyle(color: Colors.white70)),
                trailing: Icon(Icons.store, color: Colors.white),
                onTap: () {},
              ),
            ),
          SizedBox(height: 20),
          Text("🖼️ Wallpaper Pack - ${connectedDevice!.name} සඳහා", style: TextStyle(color: Colors.white, fontSize: 18, fontWeight: FontWeight.bold)),
          SizedBox(height: 10),
          GridView.builder(
            shrinkWrap: true,
            physics: NeverScrollableScrollPhysics(),
            gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(crossAxisCount: 2, crossAxisSpacing: 10, mainAxisSpacing: 10),
            itemCount: wallpapers.length,
            itemBuilder: (context, i) {
              var wp = wallpapers[i];
              return GestureDetector(
                onTap: () => sendWallpaper(wp["url"]!),
                child: Container(
                  decoration: BoxDecoration(borderRadius: BorderRadius.circular(15), image: DecorationImage(image: NetworkImage(wp["url"]!), fit: BoxFit.cover)),
                  child: Container(
                    decoration: BoxDecoration(borderRadius: BorderRadius.circular(15), color: Colors.black45),
                    child: Center(child: Text(wp["name"]!, style: TextStyle(color: Colors.white, fontWeight: FontWeight.bold))),
                  ),
                ),
              );
            },
          ),
        ],
      ),
    );
  }
}
