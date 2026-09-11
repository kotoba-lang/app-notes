(ns app-notes.model-test
  (:require [app-notes.model :as model]
            [app-notes.page :as page]
            [app-notes.source :as source]
            [clojure.test :refer [deftest is testing]]
            [design-quality.audit :as dq]
            [mokuroku.catalog :as catalog]))

(def entries
  [{:id "n1" :body "Groceries\nmilk\neggs" :folder "Home" :created 100 :modified 900}
   {:id "n2" :body "  \n\nADR review\nread 2608035000" :folder "Work"
    :created 200 :modified 500 :pinned? true}
   {:id "n3" :body "" :folder "Home" :created 300 :modified 700}])

(defn- cat-of [es]
  (catalog/refresh (catalog/catalog (source/fixture-source "All Notes" es)
                                    model/default-query)))

(deftest a-title-is-derived-not-stored
  ;; A stored title drifts: the user edits the first line and the list keeps
  ;; showing what they typed last week. Deriving makes that impossible.
  (is (= "Groceries" (model/derive-title "Groceries\nmilk")))
  (testing "leading blank lines are skipped"
    (is (= "ADR review" (model/derive-title "  \n\nADR review\nread it"))))
  (testing "an empty note is named, not blank"
    (is (= model/untitled (model/derive-title "")))
    (is (= model/untitled (model/derive-title "   \n  ")))))

(deftest the-preview-is-the-second-line
  ;; Repeating the title under the title wastes the row.
  (is (= "milk" (model/preview-line "Groceries\nmilk\neggs")))
  (is (= "" (model/preview-line "Only one line")))
  (is (= "read 2608035000" (model/preview-line "  \n\nADR review\nread 2608035000"))))

(deftest words-are-words
  (is (= 3 (model/word-count "Groceries\nmilk\neggs")))
  (is (zero? (model/word-count "")))
  (is (zero? (model/word-count "   \n  ")) "whitespace only is zero, not one"))

(deftest recently-edited-first
  (is (= ["n1" "n3" "n2"]
         (mapv :item/id (:result/items (catalog/result (cat-of entries)))))))

(deftest pinning-is-a-view-decision-not-a-sort-decision
  ;; Baking pinned-first into the comparator means a user who sorts by title
  ;; still gets pinned notes first, which is not what they asked for.
  (let [by-title (catalog/sort-by-attribute (cat-of entries) :title)
        ids (mapv :item/id (:result/items (catalog/result by-title)))]
    (is (= "n2" (first ids)) "ADR review sorts first by title, and happens to be pinned")
    (let [reversed (mapv :item/id (:result/items
                                   (catalog/result (catalog/sort-by-attribute by-title :title))))]
      (is (not= "n2" (first reversed))
          "descending really does move the pinned note off the top")))
  (is (= ["n2"] (mapv :item/id (model/pinned (model/listing->items entries))))))

(deftest notes-may-be-deleted-because-the-app-owns-the-store
  (let [c (catalog/select (cat-of entries) "n1")
        p (catalog/propose c :trash)]
    (is (true? (:proposal/destructive? p)))
    (is (= "fs/app-data" (:proposal/capability p))
        "the app's private store, not the user's filesystem")))

(deftest a-first-run-is-not-a-failure
  (is (source/first-run? (source/granted [])))
  (is (not (source/first-run? source/denied)))
  (is (not (source/first-run? (source/granted entries)))))

(deftest window-meets-the-design-quality-floor
  (let [pages {"notes" (page/render (cat-of entries))
               "selection" (page/render (catalog/select-all (cat-of entries)))
               "first-run" (page/render (catalog/refresh
                                         (catalog/catalog
                                          (source/fixture-source "All Notes" [])
                                          model/default-query)))}
        {:keys [overall pages] :as report} (dq/audit pages {:extra-axes dq/extra-axes})]
    (println "design-quality: aggregate" overall)
    (is (>= overall 98.0) (pr-str (:findings report)))
    (doseq [[nm r] pages] (is (>= (:overall r) 98.0) nm))))
