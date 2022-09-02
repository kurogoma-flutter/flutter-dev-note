# Cloud_Firestore

```dart
final cloudFirestoreServiceProvider = Provider<CloudFirestoreService>(
  (_) => CloudFirestoreService(),
);

class CloudFirestoreService {
  final FirebaseFirestore _firebaseFirestore = FirebaseFirestore.instance;

  CollectionReference<Map<String, dynamic>> getCollectionReference(
          String path) =>
      _firebaseFirestore.collection(path);

  Future<void> addData({
    required String path,
    required Map<String, dynamic> data,
  }) async {
    final reference = _firebaseFirestore.collection(path);
    await reference.add(data);
  }

  Future<void> arrayRemove({
    required String path,
    required String field,
    required dynamic elementToBeRemoved,
  }) async {
    final reference = _firebaseFirestore.doc(path);
    await reference.update(
      {
        field: FieldValue.arrayRemove([elementToBeRemoved])
      },
    );
    debugPrint('arrayRemove: $path of $field field');
  }

  Future<void> deleteField(
      {required String path, required String field}) async {
    final reference = _firebaseFirestore.doc(path);
    debugPrint('delete: $path');
    await reference.update({field: FieldValue.delete()});
  }

  Future<void> setData({
    required String path,
    required Map<String, dynamic> data,
    bool merge = true,
  }) async {
    final reference = _firebaseFirestore.doc(path);
    await reference.set(
      data,
      SetOptions(merge: merge),
    );
  }

  Future<void> updateData({
    required String path,
    required Map<String, dynamic> data,
  }) async {
    final reference = FirebaseFirestore.instance.doc(path);
    await reference.update(data);
  }

  Future<void> deleteData({required String path}) async {
    final reference = _firebaseFirestore.doc(path);
    await reference.delete();
  }

  Stream<List<T>> collectionStream<T>({
    required String path,
    required T Function(Map<String, dynamic> data, String documentID) builder,
    Query<Map<String, dynamic>> Function(Query<Map<String, dynamic>> query)?
        queryBuilder,
    int Function(T lhs, T rhs)? sort,
  }) {
    Query<Map<String, dynamic>> query = _firebaseFirestore.collection(path);
    if (queryBuilder != null) {
      query = queryBuilder(query);
    }
    final snapshots = query.snapshots();
    return snapshots.map((snapshot) {
      final result = snapshot.docs
          .map(
            (snapshot) => builder(snapshot.data(), snapshot.id),
          )
          .where((value) => value != null)
          .toList();
      if (sort != null) {
        result.sort(sort);
      }
      return result;
    });
  }

  Future<List<T>> collectionFuture<T>({
    required String path,
    required T Function(Map<String, dynamic> data, String documentID) builder,
    Query<Map<String, dynamic>> Function(Query<Map<String, dynamic>> query)?
        queryBuilder,
  }) async {
    Query<Map<String, dynamic>> query = _firebaseFirestore.collection(path);
    if (queryBuilder != null) {
      query = queryBuilder(query);
    }
    final querySnapshot = await query.get();
    return querySnapshot.docs
        .map(
          (snapshot) => builder(snapshot.data(), snapshot.id),
        )
        .where((value) => value != null)
        .toList();
  }

  Stream<T> documentStream<T>({
    required String path,
    required T Function(Map<String, dynamic>? data, String documentID) builder,
  }) {
    final reference = _firebaseFirestore.doc(path);
    final snapshots = reference.snapshots();
    return snapshots.map(
      (snapshot) => builder(snapshot.data(), snapshot.id),
    );
  }

  Future<DocumentSnapshot<Map<String, dynamic>>> fetchDocumentSnapshot({
    required String path,
  }) {
    final reference = _firebaseFirestore.doc(path);
    return reference.get();
  }

  Future<bool> hasDocumentSnapshotExisted({
    required String path,
  }) async {
    final reference = await _firebaseFirestore.doc(path).get();
    return reference.exists;
  }
}
```

# Cloud_Storage
```dart
final firebaseStorageServiceProvider =
    Provider<FirebaseStorageService>((ref) => FirebaseStorageService());

class FirebaseStorageService {
  final FirebaseStorage _firebaseStorage = FirebaseStorage.instance;

  Future<String> upload(String path, Uint8List file) async {
    final storageRef = _firebaseStorage.ref().child(path);
    UploadTask uploadTask;
    try {
      // XFileからUint8Listへ変換してCloudStorageへアップロード
      uploadTask = storageRef.putData(file);
      final snapshot = await Future.value(uploadTask);
      return await snapshot.ref.getDownloadURL();
    } catch (e) {
      return e.toString();
    }
  }

  /// file_pickerによって取得した[PlatformFile]をFirebaseStorageにアップロードし、downloadURLを返す。
  Future<String> uploadPlatformFile(
      String path, PlatformFile platformFile) async {
    final storageRef = _firebaseStorage.ref().child(path);
    final bytes = platformFile.bytes;
    if (bytes == null) {
      throw Exception('Unable to read the file');
    }

    UploadTask uploadTask;
    try {
      // XFileからUint8Listへ変換してCloudStorageへアップロード
      uploadTask = storageRef.putData(bytes);
      final snapshot = await Future.value(uploadTask);
      return await snapshot.ref.getDownloadURL();
    } catch (e) {
      return e.toString();
    }
  }
}
```

# Firebase Auth
```dart
final authStateProvider = StreamProvider<User?>((_) {
  return FirebaseAuth.instance.authStateChanges();
});

final firebaseAuthServiceProvider =
    Provider<FirebaseAuthService>((_) => FirebaseAuthService());

class FirebaseAuthService {
  final FirebaseAuth _firebaseAuth = FirebaseAuth.instance;

  Future<void> createNewFirebaseUser({
    required String email,
    required String password,
  }) async {
    await _firebaseAuth.createUserWithEmailAndPassword(
      email: email,
      password: password,
    );
    await sendEmailVerification();
  }

  Future<void> updateEmailAndPassword(
      {required String email, required String password}) async {
    await _firebaseAuth.currentUser!.updateEmail(email);
    await _firebaseAuth.currentUser!.updatePassword(password);
  }

  Future<UserCredential> signInWithEmailAndPassword({
    required String email,
    required String password,
  }) async {
    final result = await _firebaseAuth.signInWithEmailAndPassword(
        email: email, password: password);
    return result;
  }

  Future<void> get signOut async => await _firebaseAuth.signOut();

  Future<void> sendPasswordResetEmail(String email) async =>
      await _firebaseAuth.sendPasswordResetEmail(email: email);

  Future<void> sendEmailVerification() async =>
      await _firebaseAuth.currentUser!.sendEmailVerification();

  Future<void> updateEmailAddress({
    required String newEmailAddress,
    required String password,
  }) async {
    final emailAuthCredential = EmailAuthProvider.credential(
        email: currentUserEmailAddress!, password: password);
    await _firebaseAuth.currentUser!
        .reauthenticateWithCredential(emailAuthCredential);
    await _firebaseAuth.currentUser!.updateEmail(newEmailAddress);
  }

  Future<void> updatePassword(
      {required String currentPassword, required String newPassword}) async {

    final emailAuthCredential = EmailAuthProvider.credential(
      email: currentUserEmailAddress!,
      password: currentPassword,
    );
    await _firebaseAuth.currentUser!
        .reauthenticateWithCredential(emailAuthCredential);
    await _firebaseAuth.currentUser!.updatePassword(newPassword);
  }
}
```