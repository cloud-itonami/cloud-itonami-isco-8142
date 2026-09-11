(ns plasticcoord.governor-test
  (:require [clojure.test :refer [deftest is testing]]
            [plasticcoord.store :as store]
            [plasticcoord.advisor :as advisor]
            [plasticcoord.governor :as governor]))

(defn- fresh-store []
  (let [st (store/mem-store)]
    (store/register-molder! st {:molder-id "molder-1" :name "Aki Sato"})
    (store/register-facility! st {:facility-id "F-1" :name "Kobo Molding Plant" :max-supply-cost 2000})
    st))

(defn- op [op-kw & {:as extra}]
  (merge {:op op-kw :effect :propose :facility-id "F-1"
          :confidence 0.9 :stake :low}
         extra))

(def ^:private req {:molder-id "molder-1"})

(deftest ok-log-work-record
  (let [st (fresh-store)
        v (governor/check req {} (op :log-work-record) st)]
    (is (:ok? v))))

(deftest ok-schedule-crew-operation
  (let [st (fresh-store)
        v (governor/check req {} (op :schedule-crew-operation) st)]
    (is (:ok? v))))

(deftest ok-supply-order-at-threshold-boundary
  (testing "the supply-cost threshold escalate boundary is exclusive (over, not at)"
    (let [st (fresh-store)
          v (governor/check req {} (op :coordinate-supply-order :cost 2000) st)]
      (is (:ok? v)))))

(deftest hard-on-unregistered-molder
  (let [st (fresh-store)
        v (governor/check {:molder-id "nobody"} {} (op :log-work-record) st)]
    (is (:hard? v))
    (is (some #(= :no-molder (:rule %)) (:violations v)))))

(deftest hard-on-unregistered-facility
  (let [st (fresh-store)
        v (governor/check req {} (op :log-work-record :facility-id "F-ghost") st)]
    (is (:hard? v))
    (is (some #(= :no-facility (:rule %)) (:violations v)))))

(deftest hard-on-no-actuation-violation
  (let [st (fresh-store)
        v (governor/check req {} (assoc (op :log-work-record) :effect :direct-write) st)]
    (is (:hard? v))
    (is (some #(= :no-actuation (:rule %)) (:violations v)))))

(deftest hard-on-op-outside-closed-allowlist
  (let [st (fresh-store)
        v (governor/check req {} (op :dispatch-equipment) st)]
    (is (:hard? v))
    (is (some #(= :unknown-op (:rule %)) (:violations v)))))

(deftest hard-on-scope-excluded-op-finalize-molding-decision
  (testing "finalizing a molding-operation-execution decision is a permanent block, never a routine op"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (op :finalize-molding-decision) :confidence 0.99) st)]
      (is (:hard? v))
      (is (some #(= :scope-excluded-action (:rule %)) (:violations v))))))

(deftest hard-on-scope-excluded-op-finalize-molding-operation
  (testing "finalizing the molding operation is a permanent block, never a routine op"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (op :finalize-molding-operation) :confidence 0.99) st)]
      (is (:hard? v))
      (is (some #(= :scope-excluded-action (:rule %)) (:violations v))))))

(deftest hard-on-scope-excluded-op-finalize-extrusion-operation
  (testing "finalizing the extrusion operation is a permanent block, never a routine op"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (op :finalize-extrusion-operation) :confidence 0.99) st)]
      (is (:hard? v))
      (is (some #(= :scope-excluded-action (:rule %)) (:violations v))))))

(deftest hard-on-scope-excluded-op-authorize-molding-run
  (testing "authorizing a molding run is a permanent block, never a routine op"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (op :authorize-molding-run) :confidence 0.99) st)]
      (is (:hard? v))
      (is (some #(= :scope-excluded-action (:rule %)) (:violations v))))))

(deftest hard-on-scope-excluded-op-authorize-extrusion-run
  (testing "authorizing an extrusion run is a permanent block, never a routine op"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (op :authorize-extrusion-run) :confidence 0.99) st)]
      (is (:hard? v))
      (is (some #(= :scope-excluded-action (:rule %)) (:violations v))))))

(deftest hard-on-scope-excluded-op-proceed-with-molding-run
  (testing "proceeding with a molding run is a permanent block, never a routine op"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (op :proceed-with-molding-run) :confidence 0.99) st)]
      (is (:hard? v))
      (is (some #(= :scope-excluded-action (:rule %)) (:violations v))))))

(deftest hard-on-scope-excluded-op-proceed-with-extrusion-run
  (testing "proceeding with an extrusion run is a permanent block, never a routine op"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (op :proceed-with-extrusion-run) :confidence 0.99) st)]
      (is (:hard? v))
      (is (some #(= :scope-excluded-action (:rule %)) (:violations v))))))

(deftest hard-on-scope-excluded-op-finalize-plant-safety-clearance
  (testing "finalizing a plant-safety-clearance decision is a permanent block, never a routine op"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (op :finalize-plant-safety-clearance) :confidence 0.99) st)]
      (is (:hard? v))
      (is (some #(= :scope-excluded-action (:rule %)) (:violations v))))))

(deftest hard-on-scope-excluded-op-declare-plant-safety-cleared
  (testing "declaring the plant safety cleared is a permanent block, never a routine op"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (op :declare-plant-safety-cleared) :confidence 0.99) st)]
      (is (:hard? v))
      (is (some #(= :scope-excluded-action (:rule %)) (:violations v))))))

(deftest hard-on-scope-excluded-op-declare-batch-safe-for-handling
  (testing "declaring a batch safe for handling is a permanent block, never a routine op"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (op :declare-batch-safe-for-handling) :confidence 0.99) st)]
      (is (:hard? v))
      (is (some #(= :scope-excluded-action (:rule %)) (:violations v))))))

(deftest hard-on-scope-excluded-op-declare-molded-batch-safe-for-handling
  (testing "declaring a molded batch safe for handling (alternate op name) is a permanent block, never a routine op"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (op :declare-molded-batch-safe-for-handling) :confidence 0.99) st)]
      (is (:hard? v))
      (is (some #(= :scope-excluded-action (:rule %)) (:violations v))))))

(deftest hard-on-scope-excluded-op-declare-batch-safe-for-shipment
  (testing "declaring a batch safe for shipment is a permanent block, never a routine op"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (op :declare-batch-safe-for-shipment) :confidence 0.99) st)]
      (is (:hard? v))
      (is (some #(= :scope-excluded-action (:rule %)) (:violations v))))))

(deftest hard-on-scope-excluded-op-clear-batch-for-shipment
  (testing "clearing a batch for shipment is a permanent block, never a routine op"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (op :clear-batch-for-shipment) :confidence 0.99) st)]
      (is (:hard? v))
      (is (some #(= :scope-excluded-action (:rule %)) (:violations v))))))

(deftest hard-on-scope-excluded-op-override-plant-safety-officer-judgment
  (testing "overriding a plant safety officer's judgment is a plant safety officer's exclusive judgment, never this actor's"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (op :override-plant-safety-officer-judgment) :confidence 0.99) st)]
      (is (:hard? v))
      (is (some #(= :scope-excluded-action (:rule %)) (:violations v))))))

(deftest hard-on-scope-excluded-rationale-molding-run
  (testing "defense-in-depth: a rationale that itself attempts to proceed with the molding run is blocked even if the op looks routine"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (op :schedule-crew-operation)
                                           :rationale "recommend we proceed with the molding run now")
                             st)]
      (is (:hard? v))
      (is (some #(= :scope-excluded-action (:rule %)) (:violations v))))))

(deftest hard-on-scope-excluded-rationale-extrusion-run
  (testing "defense-in-depth: a rationale that itself attempts to proceed with the extrusion run is blocked even if the op looks routine"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (op :schedule-crew-operation)
                                           :rationale "recommend we proceed with the extrusion run now")
                             st)]
      (is (:hard? v))
      (is (some #(= :scope-excluded-action (:rule %)) (:violations v))))))

(deftest hard-on-scope-excluded-rationale-plant-safety-cleared
  (testing "defense-in-depth: a rationale that itself attempts to declare the plant safety cleared is blocked even if the op looks routine"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (op :log-work-record)
                                           :rationale "we can declare the plant safety cleared now")
                             st)]
      (is (:hard? v))
      (is (some #(= :scope-excluded-action (:rule %)) (:violations v))))))

(deftest hard-on-scope-excluded-rationale-finalize-molding-operation
  (testing "defense-in-depth: a rationale that itself attempts to finalize the molding operation is blocked even if the op looks routine"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (op :coordinate-supply-order :cost 100)
                                           :rationale "recommend we finalize the molding operation today")
                             st)]
      (is (:hard? v))
      (is (some #(= :scope-excluded-action (:rule %)) (:violations v))))))

(deftest hard-on-scope-excluded-rationale-override-plant-safety-officer
  (testing "defense-in-depth: a rationale attempting to override the plant safety officer's judgment is blocked even if the op looks routine"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (op :log-work-record)
                                           :rationale "override the plant safety officer's judgment and proceed")
                             st)]
      (is (:hard? v))
      (is (some #(= :scope-excluded-action (:rule %)) (:violations v))))))

(deftest hard-on-scope-excluded-rationale-declare-batch-safe-for-handling
  (testing "defense-in-depth: a rationale that itself attempts to declare the batch safe for handling is blocked even if the op looks routine"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (op :log-work-record)
                                           :rationale "we should declare the molded batch safe for handling")
                             st)]
      (is (:hard? v))
      (is (some #(= :scope-excluded-action (:rule %)) (:violations v))))))

(deftest always-escalates-safety-concern-even-at-high-confidence
  (testing "a machinery-hazard [crush/entanglement]/heat-exposure/fume-exposure/equipment-condition concern always requires human sign-off"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (op :flag-safety-concern :hazard-type :machinery-hazard)
                                           :confidence 0.99)
                             st)]
      (is (not (:hard? v)))
      (is (:escalate? v)))))

(deftest always-escalates-supply-order-above-threshold
  (let [st (fresh-store)
        v (governor/check req {} (assoc (op :coordinate-supply-order :cost 5000) :confidence 0.99) st)]
    (is (not (:hard? v)))
    (is (:escalate? v))))

(deftest escalates-low-confidence
  (let [st (fresh-store)
        v (governor/check req {} (assoc (op :log-work-record) :confidence 0.3) st)]
    (is (not (:hard? v)))
    (is (:escalate? v))))

(deftest default-mock-advisor-proposals-never-self-trip-on-scope-exclusion
  (testing "the governor's scope-exclusion term list must never match the mock advisor's own default rationale text for any allowlisted op — CLAUDE.md's known self-tripping bug pattern (rationale legitimately contains bare nouns like 'plastic'/'resin'/'extruder'/'molding', but never the full finalization-action phrases)"
    (let [st (fresh-store)
          adv (advisor/mock-advisor)
          ops [:log-work-record :schedule-crew-operation
               :flag-safety-concern :coordinate-supply-order]]
      (doseq [o ops]
        (let [request {:molder-id "molder-1" :op o :facility-id "F-1"
                        :stake :low :task "routine injection molding task with plastic resin at the extruder"
                        :hazard-type :machinery-hazard :cost 500}
              proposal (advisor/-advise adv st request)
              v (governor/check request {} proposal st)]
          (is (not (:hard? v))
              (str o " proposal unexpectedly hard-blocked: " (:violations v))))))))
