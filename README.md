import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:google_maps_flutter/google_maps_flutter.dart';
import 'package:geolocator/geolocator.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: const HomePage(),
    );
  }
}

class HomePage extends StatefulWidget {
  const HomePage({super.key});
  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  final FirebaseAuth _auth = FirebaseAuth.instance;
  final FirebaseFirestore _db = FirebaseFirestore.instance;
  User? user;
  GoogleMapController? mapController;
  LatLng? currentPosition;
  final Set<Marker> _markers = {};

  @override
  void initState() {
    super.initState();
    _auth.authStateChanges().listen((u) {
      setState(() {
        user = u;
      });
    });
    _determinePosition();
  }

  Future<void> _determinePosition() async {
    LocationPermission permission;
    permission = await Geolocator.requestPermission();
    Position pos = await Geolocator.getCurrentPosition(
        desiredAccuracy: LocationAccuracy.high);
    setState(() {
      currentPosition = LatLng(pos.latitude, pos.longitude);
      _markers.add(Marker(
        markerId: const MarkerId('current'),
        position: currentPosition!,
        infoWindow: const InfoWindow(title: 'مكاني الحالي'),
      ));
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('وصلناك - طلب سيارة'),
        backgroundColor: Colors.blueAccent,
        actions: [
          if (user != null)
            IconButton(
                icon: const Icon(Icons.logout),
                onPressed: () async {
                  await _auth.signOut();
                })
        ],
      ),
      body: currentPosition == null
          ? const Center(child: CircularProgressIndicator())
          : GoogleMap(
              initialCameraPosition:
                  CameraPosition(target: currentPosition!, zoom: 15),
              markers: _markers,
              onMapCreated: (controller) => mapController = controller,
            ),
      floatingActionButton: FloatingActionButton(
        child: const Icon(Icons.local_taxi),
        onPressed: _requestRide,
      ),
    );
  }

  void _requestRide() async {
    if (user == null) {
      _showMessage('الرجاء تسجيل الدخول أولاً');
      return;
    }
    if (currentPosition == null) return;

    await _db.collection('rides').add({
      'userId': user!.uid,
      'location': {
        'lat': currentPosition!.latitude,
        'lng': currentPosition!.longitude
      },
      'status': 'requested',
      'timestamp': FieldValue.serverTimestamp(),
    });

    _showMessage('تم طلب السيارة بنجاح!');
  }

  void _showMessage(String msg) {
    ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(msg)));
  }
}
