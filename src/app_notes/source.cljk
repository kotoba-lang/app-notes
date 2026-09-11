(ns app-notes.source
  "The `fs/app-data` seam. The store is the app's own, not the user's disk."
  (:require [app-notes.model :as model]
            [mokuroku.source :as source]))

(defrecord NoteSource [folder read-fn]
  source/ISource
  (-descriptor [_] (model/descriptor folder))
  (-fetch [_] (model/listing->items (read-fn folder))))

(defn note-source [folder read-fn] (->NoteSource folder read-fn))
(defn fixture-source [folder entries] (note-source folder (constantly entries)))

(def denied
  {:notes/state :denied
   :notes/capability model/capability
   :notes/entries []})

(defn granted [entries]
  {:notes/state :granted
   :notes/capability model/capability
   :notes/entries (vec entries)})

(defn denied? [r] (= :denied (:notes/state r)))

;; Unlike a process table, a genuinely empty note store is normal: it is what
;; a new install looks like. So an empty result here is a fact, not a fault,
;; and the app says "no notes yet" rather than implying something failed.
(defn first-run? [r]
  (and (not (denied? r)) (empty? (:notes/entries r))))
