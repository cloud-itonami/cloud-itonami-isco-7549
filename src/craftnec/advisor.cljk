(ns craftnec.advisor
  "Craft Trades Coordination Advisor — proposing a craft workshop
  scheduling/logistics coordination operation (log a work record,
  schedule a crew operation, flag a safety concern, coordinate a
  craft-materials supply order) from a crew roster, workshop
  registration and safety-reporting policy. Swappable mock/llm; the
  advisor ONLY proposes — `craftnec.governor` independently gates
  every proposal and always escalates safety concerns and
  above-threshold supply orders. ISCO-08 7549 (Craft and Related
  Workers Not Elsewhere Classified) is a broad/generic residual
  craft-work category — varied hand-tools, varied materials, no
  single dominant hazard type — so hazard reporting here stays
  generic (hand-tool-hazard / material-handling-hazard /
  equipment-condition) rather than naming one dominant
  technique-specific hazard. The advisor never proposes to directly
  finalize a craft-execution decision (e.g. deciding a craft item is
  finished) or a product-quality/safety-clearance decision (e.g.
  declaring an item quality- or safety-cleared), or to override a
  shop safety officer's judgment — those stay permanently out of
  this actor's scope. Modeled closely on cloud-itonami-isco-7319's
  craftcoord.advisor (closest domain shape — a published, tested
  generic workshop scheduling/logistics coordination pattern for the
  sibling residual craft-work category).

  A proposal: {:op :log-work-record|:schedule-crew-operation|
               :flag-safety-concern|:coordinate-supply-order
               :effect :propose :worker-id str :workshop-id str
               :cost number :hazard-type kw :task str :stake kw
               :confidence n :rationale str}")

(defprotocol Advisor
  (-advise [advisor store request] "request -> proposal map"))

(defn- rationale-for [op worker-id workshop-id hazard-type]
  (case op
    :log-work-record
    (str "logged work record for worker " worker-id " at workshop " workshop-id)

    :schedule-crew-operation
    (str "scheduled crew operation for craft task at workshop " workshop-id)

    :flag-safety-concern
    (str "flagged " (name (or hazard-type :hazard)) " concern for worker "
         worker-id " at workshop " workshop-id " — routed for shop safety officer review")

    :coordinate-supply-order
    (str "coordinated supply order for worker " worker-id " at workshop " workshop-id)

    (str "proposed " (name op) " for worker " worker-id " at workshop " workshop-id)))

(defn- infer [_store {:keys [op stake worker-id workshop-id cost hazard-type task]
                       :as request}]
  {:op op
   :effect :propose
   :worker-id worker-id
   :workshop-id workshop-id
   :cost cost
   :hazard-type hazard-type
   :task task
   :stake (or stake :low)
   :confidence (case (or stake :low) :high 0.7 :medium 0.85 :low 0.95)
   :rationale (rationale-for op worker-id workshop-id hazard-type)})

(defn mock-advisor []
  (reify Advisor
    (-advise [_ store request] (infer store request))))

(def ^:private system-prompt
  "You are a craft workshop scheduling/logistics coordination
   advisor covering ISCO-08 7549 (Craft and Related Workers Not
   Elsewhere Classified) — a broad, generic residual craft-work
   category with varied hand-tools and varied materials and no
   single dominant hazard type. Given a request, propose an :op (one
   of :log-work-record, :schedule-crew-operation,
   :flag-safety-concern, :coordinate-supply-order), the :worker-id,
   :workshop-id, and any :cost/:hazard-type/:task fields, an honest
   :confidence and a :stake. Never propose an op outside this closed
   list, and never propose to directly finalize a craft-execution
   decision (e.g. deciding a craft item is finished) or a
   product-quality/safety-clearance decision (e.g. declaring an item
   quality- or safety-cleared), or to override a shop safety
   officer's judgment — those are always out of this actor's scope;
   it coordinates workshop scheduling/logistics only and never
   performs craft work or makes quality/safety-clearance decisions
   itself. Safety concerns always require human sign-off regardless
   of confidence.")

(defn- parse-proposal [content]
  (try
    (let [p (read-string content)]
      (if (map? p)
        (assoc p :effect :propose)
        {:op :unknown :effect :propose :confidence 0.0 :stake :high
         :rationale "unparseable LLM response"}))
    (catch #?(:clj Exception :cljs js/Error) _
      {:op :unknown :effect :propose :confidence 0.0 :stake :high
       :rationale "LLM response parse failure"})))

(defn llm-advisor
  [chat-model model-generate-fn gen-opts]
  (reify Advisor
    (-advise [_ _store request]
      (let [msgs [{:role :system :content system-prompt}
                  {:role :user :content (str "operation request: " (pr-str request))}]
            resp (model-generate-fn chat-model msgs gen-opts)]
        (parse-proposal (:content resp))))))
