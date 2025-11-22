import Foundation

// MARK: - Models
enum LedgerEventType: String, Codable {
    case deposit, withdraw
}

struct LedgerEvent: Codable {
    let id: UUID
    let type: LedgerEventType
    let amount: Double
    let timestamp: Date
}

// MARK: - Event Store
class EventStore {
    private var events: [LedgerEvent] = []
    
    func append(type: LedgerEventType, amount: Double) {
        events.append(
            LedgerEvent(id: UUID(), type: type, amount: amount, timestamp: Date())
        )
    }
    
    func all() -> [LedgerEvent] { events }
}

// MARK: - Ledger Engine
class LedgerEngine {
    private let store = EventStore()
    
    func deposit(_ amount: Double) { store.append(type: .deposit, amount: amount) }
    func withdraw(_ amount: Double) { store.append(type: .withdraw, amount: amount) }
    
    func snapshotBalance() -> Double {
        return store.all().reduce(0) { sum, event in
            switch event.type {
            case .deposit: return sum + event.amount
            case .withdraw: return sum - event.amount
            }
        }
    }
    
    func printHistory() {
        print("=== Ledger History ===")
        store.all().forEach {
            print("\($0.timestamp): \($0.type.rawValue.uppercased()) \($0.amount)")
        }
        print("======================")
    }
}

// MARK: - Demo
let ledger = LedgerEngine()

ledger.deposit(500)
ledger.withdraw(120)
ledger.deposit(80)

ledger.printHistory()
print("Balance:", ledger.snapshotBalance())
