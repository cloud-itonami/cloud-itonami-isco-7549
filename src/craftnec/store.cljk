(ns craftnec.store
  "SSoT for the ISCO-08 7549 craft workshop scheduling/logistics
  coordination actor (itonami actor pattern, ADR-2607121000 /
  CLAUDE.md Actors section; README's 'Robotics premise' — a workshop
  scheduling/logistics coordination robot performs crew scheduling,
  job/commission/progress-record logging and craft-materials
  supply-order coordination for a craft workshop under this
  advisor/governor pair, which never dispatches hardware itself,
  never performs craft work itself, and never finalizes a
  craft-execution decision or a product-quality/safety-clearance
  decision, nor overrides a shop safety officer's judgment — those
  remain the shop safety officer's exclusive judgment). ISCO-08 7549
  (Craft and Related Workers Not Elsewhere Classified) is a
  broad/generic residual craft-work category — varied hand-tools,
  varied materials, no single dominant technique or hazard type — so
  this store's domain shape stays generic (a job/commission record,
  not a technique-specific batch record). Modeled closely on
  cloud-itonami-isco-7319's craftcoord.store (closest domain shape —
  a published, tested generic workshop scheduling/logistics
  coordination pattern for the sibling residual craft-work category).

  Domain:

    worker   — a registered craft workshop crew member (:worker-id,
               :name)
    workshop — a registered craft workshop site {:workshop-id :name
               :max-supply-cost}. `:max-supply-cost` is an
               informational registered ceiling used only to decide
               whether a `:coordinate-supply-order` proposal escalates
               to human sign-off (the governor never blocks a
               within-threshold order outright; it only decides
               commit vs. escalate).
    record   — a committed operating record (a logged job/commission/
               progress entry, a scheduled crew/task operation, a
               flagged safety concern, or a coordinated craft-materials
               supply order) — written ONLY via commit-record!.
    ledger   — append-only audit trail, commit or hold.")

(defprotocol Store
  (worker [s worker-id])
  (workshop [s workshop-id])
  (records-of [s worker-id])
  (ledger [s])
  (register-worker! [s w])
  (register-workshop! [s w])
  (commit-record! [s record])
  (append-ledger! [s fact]))

(defrecord MemStore [a]
  Store
  (worker [_ worker-id] (get-in @a [:workers worker-id]))
  (workshop [_ workshop-id] (get-in @a [:workshops workshop-id]))
  (records-of [_ worker-id] (filter #(= worker-id (:worker-id %)) (:records @a)))
  (ledger [_] (:ledger @a))
  (register-worker! [s m]
    (swap! a assoc-in [:workers (:worker-id m)] m) s)
  (register-workshop! [s w]
    (swap! a assoc-in [:workshops (:workshop-id w)] w) s)
  (commit-record! [s record]
    (swap! a update :records (fnil conj []) record) s)
  (append-ledger! [s fact]
    (swap! a update :ledger (fnil conj []) fact) s))

(defn mem-store
  ([] (mem-store {}))
  ([seed] (->MemStore (atom (merge {:workers {} :workshops {} :records [] :ledger []}
                                    seed)))))
