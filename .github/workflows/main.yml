
import 'dart:io';
import 'dart:typed_data';
import 'package:flutter/material.dart';
import 'package:image_picker/image_picker.dart';
import 'package:image/image.dart' as img;
import 'package:gal/gal.dart';

const purple = Color(0xFF6842E8);

void main() {
  runApp(const PhotoMagicApp());
}

class PhotoMagicApp extends StatelessWidget {
  const PhotoMagicApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Photo Magic',
      theme: ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(
          seedColor: purple,
          brightness: Brightness.light,
        ),
        scaffoldBackgroundColor: const Color(0xFFF8F7FC),
        appBarTheme: const AppBarTheme(
          backgroundColor: Color(0xFFF8F7FC),
          elevation: 0,
        ),
      ),
      home: const HomePage(),
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  Future<void> openGallery(BuildContext context) async {
    final picker = ImagePicker();
    final photo = await picker.pickImage(
      source: ImageSource.gallery,
      imageQuality: 95,
    );

    if (photo == null || !context.mounted) return;

    final bytes = await photo.readAsBytes();

    if (!context.mounted) return;

    Navigator.push(
      context,
      MaterialPageRoute(
        builder: (_) => EditorPage(originalBytes: bytes),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Row(
          children: [
            Icon(Icons.auto_awesome, color: purple, size: 27),
            SizedBox(width: 9),
            Text(
              'Photo Magic',
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
          ],
        ),
      ),
      body: Padding(
        padding: const EdgeInsets.all(22),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const SizedBox(height: 25),
            const Text(
              'Make every picture\nbeautiful.',
              style: TextStyle(
                fontSize: 34,
                height: 1.2,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 12),
            const Text(
              'Your creativity, our magic. Edit your photos beautifully.',
              style: TextStyle(color: Colors.black54, fontSize: 15),
            ),
            const SizedBox(height: 35),
            Container(
              width: double.infinity,
              padding: const EdgeInsets.all(28),
              decoration: BoxDecoration(
                gradient: const LinearGradient(
                  colors: [Color(0xFF7755F3), Color(0xFF4425B8)],
                  begin: Alignment.topLeft,
                  end: Alignment.bottomRight,
                ),
                borderRadius: BorderRadius.circular(25),
              ),
              child: Column(
                children: [
                  const Icon(
                    Icons.add_photo_alternate_outlined,
                    size: 75,
                    color: Colors.white,
                  ),
                  const SizedBox(height: 18),
                  const Text(
                    'Start creating magic',
                    style: TextStyle(
                      fontSize: 21,
                      color: Colors.white,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  const SizedBox(height: 10),
                  const Text(
                    'Select a photo and transform it.',
                    style: TextStyle(color: Colors.white70),
                  ),
                  const SizedBox(height: 24),
                  SizedBox(
                    width: double.infinity,
                    child: ElevatedButton.icon(
                      onPressed: () => openGallery(context),
                      icon: const Icon(Icons.photo_library),
                      label: const Text('Choose a photo'),
                      style: ElevatedButton.styleFrom(
                        backgroundColor: Colors.white,
                        foregroundColor: purple,
                        padding: const EdgeInsets.all(16),
                      ),
                    ),
                  ),
                ],
              ),
            ),
            const SizedBox(height: 35),
            const Text(
              'Editing tools',
              style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 18),
            const Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                ToolIcon(icon: Icons.brightness_6, label: 'Brightness'),
                ToolIcon(icon: Icons.contrast, label: 'Contrast'),
                ToolIcon(icon: Icons.rotate_right, label: 'Rotate'),
                ToolIcon(icon: Icons.filter_vintage, label: 'Filters'),
              ],
            ),
          ],
        ),
      ),
    );
  }
}

class ToolIcon extends StatelessWidget {
  final IconData icon;
  final String label;

  const ToolIcon({super.key, required this.icon, required this.label});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Container(
          padding: const EdgeInsets.all(15),
          decoration: BoxDecoration(
            color: Colors.white,
            borderRadius: BorderRadius.circular(16),
          ),
          child: Icon(icon, color: purple, size: 27),
        ),
        const SizedBox(height: 8),
        Text(label, style: const TextStyle(fontSize: 11)),
      ],
    );
  }
}

class EditorPage extends StatefulWidget {
  final Uint8List originalBytes;

  const EditorPage({super.key, required this.originalBytes});

  @override
  State<EditorPage> createState() => _EditorPageState();
}

class _EditorPageState extends State<EditorPage> {
  double brightness = 1.0;
  double contrast = 1.0;
  bool blackWhite = false;
  int rotation = 0;
  Uint8List? editedBytes;
  bool saving = false;

  void applyEdits() {
    try {
      final decoded = img.decodeImage(widget.originalBytes);
      if (decoded == null) return;

      var result = img.adjustColor(
        decoded,
        brightness: brightness,
        contrast: contrast,
      );

      if (blackWhite) {
        result = img.grayscale(result);
      }

      if (rotation != 0) {
        result = img.copyRotate(result, angle: rotation);
      }

      setState(() {
        editedBytes = Uint8List.fromList(img.encodePng(result));
      });
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('Could not process this photo.')),
      );
    }
  }

  void resetEdits() {
    setState(() {
      brightness = 1.0;
      contrast = 1.0;
      blackWhite = false;
      rotation = 0;
      editedBytes = null;
    });
  }

  Future<void> savePhoto() async {
    setState(() => saving = true);

    try {
      final bytes = editedBytes ?? widget.originalBytes;
      final file = File(
        '${Directory.systemTemp.path}/photo_magic_${DateTime.now().millisecondsSinceEpoch}.png',
      );

      await file.writeAsBytes(bytes);
      await Gal.putImage(file.path);

      if (!mounted) return;

      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('Photo saved to your gallery!')),
      );
    } catch (e) {
      if (!mounted) return;

      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Could not save photo: $e')),
      );
    } finally {
      if (mounted) setState(() => saving = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        leading: const BackButton(),
        title: const Text(
          'Edit Photo',
          style: TextStyle(fontWeight: FontWeight.bold),
        ),
        actions: [
          TextButton(
            onPressed: resetEdits,
            child: const Text('Reset'),
          ),
        ],
      ),
      body: Column(
        children: [
          Expanded(
            child: Container(
              margin: const EdgeInsets.all(16),
              width: double.infinity,
              decoration: BoxDecoration(
                color: Colors.black,
                borderRadius: BorderRadius.circular(18),
              ),
              clipBehavior: Clip.antiAlias,
              child: Image.memory(
                editedBytes ?? widget.originalBytes,
                fit: BoxFit.contain,
              ),
            ),
          ),
          Container(
            padding: const EdgeInsets.fromLTRB(22, 15, 22, 22),
            decoration: const BoxDecoration(
              color: Colors.white,
              borderRadius: BorderRadius.vertical(
                top: Radius.circular(25),
              ),
            ),
            child: Column(
              children: [
                Row(
                  children: [
                    const Text(
                      'Adjustments',
                      style: TextStyle(
                        fontSize: 19,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    const Spacer(),
                    IconButton(
                      onPressed: () {
                        setState(() {
                          rotation = (rotation + 90) % 360;
                        });
                        applyEdits();
                      },
                      icon: const Icon(Icons.rotate_right, color: purple),
                      tooltip: 'Rotate',
                    ),
                  ],
                ),
                const SizedBox(height: 8),
                Row(
                  children: [
                    const Icon(Icons.brightness_6, color: purple),
                    const SizedBox(width: 10),
                    const Text('Brightness'),
                    Expanded(
                      child: Slider(
                        value: brightness,
                        min: 0.3,
                        max: 1.7,
                        activeColor: purple,
                        onChanged: (v) {
                          setState(() => brightness = v);
                          applyEdits();
                        },
                      ),
                    ),
                  ],
                ),
                Row(
                  children: [
                    const Icon(Icons.contrast, color: purple),
                    const SizedBox(width: 10),
                    const Text('Contrast'),
                    Expanded(
                      child: Slider(
                        value: contrast,
                        min: 0.3,
                        max: 2.0,
                        activeColor: purple,
                        onChanged: (v) {
                          setState(() => contrast = v);
                          applyEdits();
                        },
                      ),
                    ),
                  ],
                ),
                SwitchListTile(
                  contentPadding: EdgeInsets.zero,
                  title: const Text('Black and white filter'),
                  secondary: const Icon(Icons.filter_b_and_w, color: purple),
                  value: blackWhite,
                  activeThumbColor: purple,
                  onChanged: (v) {
                    setState(() => blackWhite = v);
                    applyEdits();
                  },
                ),
                const SizedBox(height: 12),
                SizedBox(
                  width: double.infinity,
                  child: ElevatedButton.icon(
                    onPressed: saving ? null : savePhoto,
                    icon: saving
                        ? const SizedBox(
                            width: 18,
                            height: 18,
                            child: CircularProgressIndicator(strokeWidth: 2),
                          )
                        : const Icon(Icons.download),
                    label: Text(saving ? 'Saving...' : 'Save Photo'),
                    style: ElevatedButton.styleFrom(
                      backgroundColor: purple,
                      foregroundColor: Colors.white,
                      padding: const EdgeInsets.all(17),
                    ),
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
