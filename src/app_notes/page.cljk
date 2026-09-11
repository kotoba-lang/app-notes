(ns app-notes.page
  (:require [mokuroku.catalog :as catalog]
            [mokuroku-ui.core :as mui]))

(def view-opts
  {:columns [:title :modified :words]
   :formatters {:words #(when % (str % (if (= 1 %) " word" " words")))}
   :noun "notes"
   :search-placeholder "Search notes"
   :empty-title "No notes yet"
   ;; A first run is not a failure. Saying "nothing matches" to someone who
   ;; has never written a note describes a filter they never set.
   :empty-body "Notes you write are stored privately by this app."
   :badge (fn [it] (when (:pinned (:item/attrs it)) "Pinned"))
   :title "Notes"
   :description "Your notes, most recently edited first."})

(defn render [cat] (mui/->page (catalog/view cat) view-opts))
(defn render-html [cat] (mui/->html (catalog/view cat) view-opts))
