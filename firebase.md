# Cloud_Firestore

```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';

final cloudFirestoreServiceProvider = Provider<CloudFirestoreService>(
  (_) => CloudFirestoreService(),
);

class CloudFirestoreService {
  final FirebaseFirestore _firebaseFirestore = FirebaseFirestore.instance;

  CollectionReference<Map<String, dynamic>> getCollectionReference(
          String path) =>
      _firebaseFirestore.collection(path);

  /// データを追加する
  ///
  /// [path] コレクションのパス
  ///
  /// [data] 追加するデータ
  Future<void> addData({
    required String path,
    required Map<String, dynamic> data,
  }) async {
    final reference = _firebaseFirestore.collection(path);
    await reference.add(data);
  }

  /// データを更新 or 追加する
  ///
  /// [path] コレクションのパス
  ///
  /// [data] 更新するデータ
  ///
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

  /// データを更新する
  ///
  /// [path] ドキュメントのパス
  ///
  /// [data] 更新するデータ
  ///
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

  /// コレクション単位のStreamを取得する
  ///
  /// [path] コレクションのパス
  ///
  /// [queryBuilder] クエリを設定する関数
  ///
  /// [sort] ソートするフィールド名
  ///
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

  /// コレクション単位でのデータ取得
  ///
  /// [path] コレクションのパス
  ///
  /// [builder] データを変換する関数
  ///
  /// [queryBuilder] クエリを組み立てる関数
  ///
  /// [sort] ソートする関数
  ///
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

  /// ドキュメントのストリームを取得する
  ///
  /// [path] ドキュメントのパス
  ///
  /// [builder] ドキュメントのデータを変換する関数（Map変換など）
  ///
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

  /// 単一documentを取得する
  ///
  /// [path] documentのパス
  ///
  Future<DocumentSnapshot<Map<String, dynamic>>> fetchDocumentSnapshot({
    required String path,
  }) {
    final reference = _firebaseFirestore.doc(path);
    return reference.get();
  }

  /// documentが存在するかを確認する
  ///
  /// [path] documentのpath
  ///
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