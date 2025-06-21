>[!question]
>GQ1. 이거 CloudKit 뭔지 알아보기.
>GQ2. 애플 생태계에선 Firebase를 대체할 수 있을까?

## Description
- 일단 CloudKit이 뭐냐면, iCloud 인프라를 이용해 애플 생태계 내부에서 데이터베이스를 구현하는 겁니다.
- 솔직히 쓰라는 건지 말라는 건지, 최신 정보랑 업데이트도 별로 없고 공식 샘플 코드도 너무 구린 것 같습니다.

## 주요 기능
+ Private DB ( 많이 쓰임 ) : 유저의 로컬 데이터를 유저 개인의 iCloud에 저장하거나 불러옵니다.
+ Public DB ( 가끔 쓰임 ) : 앱의 모든 사용자가 공유하는 데이터베이스입니다. 개발자는 CloudKit Console이라는 곳에서 권한을 설정하거나 내용을 볼 수도 있고, 수정도 가능하다고 합니다. 게시판등등 컨텐츠를 공유하는데에 많이 쓰인대요.
+ Shared DB ( 거의 안쓰임 ) : 유저의 로컬 데이터(iCloud 데이터)를 다른 유저와도 공유할 수 있습니다. 코드도 복잡하고 자료도 없고 실시간도 아니라 딱히 쓸데도 없대요. Keynote 팀원과 공유해서 사용해보셨으면 그 기능인 것 같습니다.

## 코드 예시
#### Private DB :
- Xcode 프로젝트 설정 > Signing & Capabilities > iCloud 추가하기
- Apple Developer Portal에서 앱 ID에 iCloud capability 등록하기
- [[Core Data]]: 코드에서 NSPersistentContainer를 NSPersistentCloudKitContainer로 변환하면 끝.
- [[Swift Data]]: 코드 변경사항 없음

#### Public DB:
- 퍼블릭 DB에 데이터 쓰기, 기본적으로 쓰기 권한은 개발자에게만 있는데 CloudKit Console이라는 곳에서 권한을 수정할 수 있습니다.
```
import CloudKit

func savePostToPublicDB(title: String, content: String) {
    let record = CKRecord(recordType: "Post")
    record["title"] = title as CKRecordValue
    record["content"] = content as CKRecordValue
    record["createdAt"] = Date() as CKRecordValue

    let publicDB = CKContainer.default().publicCloudDatabase

    publicDB.save(record) { savedRecord, error in
        if let error = error {
            print("❌ 저장 실패: \(error)")
        } else {
            print("✅ 저장 성공: \(savedRecord!.recordID)")
        }
    }
}
```
-  퍼블릭 DB에서 데이터 읽기
```
func fetchAllPosts(completion: @escaping ([CKRecord]) -> Void) {
    let publicDB = CKContainer.default().publicCloudDatabase
    let query = CKQuery(recordType: "Post", predicate: NSPredicate(value: true)) // 모든 Post 가져오기

    publicDB.perform(query, inZoneWith: nil) { records, error in
        if let error = error {
            print("❌ 조회 실패: \(error)")
            completion([])
        } else {
            print("✅ 조회 성공: \(records?.count ?? 0)개")
            completion(records ?? [])
        }
    }
}
```

#### Shared DB: 읽지 않으셔도 됩니다.
- 공유가능한 객체(CKRecord)만들기
```
let record = CKRecord(recordType: "Note")
record["title"] = "공유 테스트"
record["content"] = "이건 공유된 노트입니다."

let privateDB = CKContainer.default().privateCloudDatabase

privateDB.save(record) { savedRecord, error in
    if let error = error {
        print("❌ Save 실패: \(error)")
    } else if let savedRecord = savedRecord {
        print("✅ Record 저장됨: \(savedRecord.recordID)")
    }
}
```
- 공유(CKShare)만들기
```
let share = CKShare(rootRecord: record)
share[CKShare.SystemFieldKey.title] = "공유된 노트" as CKRecordValue

let modifyOp = CKModifyRecordsOperation(recordsToSave: [record, share], recordIDsToDelete: nil)
modifyOp.modifyRecordsResultBlock = { result in
    switch result {
    case .success:
        print("✅ 공유 생성 성공")
    case .failure(let error):
        print("❌ 공유 생성 실패: \(error)")
    }
}

CKContainer.default().privateCloudDatabase.add(modifyOp)
```
- 애플 제공 공유 UI
```
import UIKit
import CloudKit

func showSharingUI(from viewController: UIViewController, share: CKShare, container: CKContainer) {
    let controller = UICloudSharingController(share: share, container: container)
    controller.delegate = /* 공유 상태 변화 감지용 델리게이트 */
    viewController.present(controller, animated: true)
}
```
- 공유된 데이터 수신하기
```
let sharedDB = CKContainer.default().sharedCloudDatabase
let query = CKQuery(recordType: "Note", predicate: NSPredicate(value: true))

sharedDB.perform(query, inZoneWith: nil) { results, error in
    if let results = results {
        for record in results {
            print("📄 공유된 노트: \(record["title"] ?? "")")
        }
    }
}
```
## Keywords
+ [[Swift Data]]
+ [[Core Data]]

## References
- https://developer.apple.com/kr/icloud/cloudkit/designing/
- https://github.com/apple?q=sample-cloudkit&type=repository