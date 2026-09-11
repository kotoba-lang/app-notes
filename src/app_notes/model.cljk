(ns app-notes.model
  "Notes' domain: the app's own documents.

  One capability, and a narrow one: `fs/app-data` is the app's private store,
  not the user's filesystem. A notes app has no business reading Documents —
  it reads and writes the notes it created. This is the same capability
  aiueos's `examples/apps/notes.edn` requests, and for the same reason."
  (:require [kotoba.lang.text :as str]
            [mokuroku.item :as item]
            [mokuroku.source :as source]))

(def capability "fs/app-data")

(def columns
  [(source/attribute :title "Title" :string)
   (source/attribute :modified "Date Modified" :number)
   (source/attribute :folder "Folder" :string)
   (source/attribute :words "Words" :number)
   (source/attribute :created "Date Created" :number)
   (source/attribute :preview "Preview" :string false)])

(def commands
  "Notes are the one thing in this suite the app itself owns, so unlike a file
  listing it can offer deletion — the store it deletes from is its own."
  #{:open :rename :trash :export :copy-path})

(defn descriptor
  ([] (descriptor "All Notes"))
  ([folder]
   (source/descriptor
    {:id :app-notes/notes
     :item-kind :note
     :label folder
     :capability capability
     :commands commands
     :attributes columns})))

(def untitled "New Note")

(defn derive-title
  "A note's title is its first non-blank line, not a separate field.

  Keeping a title field beside the body means the two drift: the user edits
  the first line and the list still shows what they typed a week ago. Deriving
  it makes that impossible."
  [body]
  (or (->> (str/split-lines (str body))
           (map str/trim)
           (remove str/blank?)
           first)
      untitled))

(defn word-count
  "Words, not characters. Whitespace-separated runs, so an empty note is 0 and
  a note of only whitespace is also 0."
  [body]
  (count (remove str/blank? (str/split (str body) #"\s+"))))

(defn preview-line
  "The second non-blank line, which is what the list shows under the title.
  Repeating the title there wastes the row."
  [body]
  (or (->> (str/split-lines (str body))
           (map str/trim)
           (remove str/blank?)
           second)
      ""))

(defn entry->item
  [{:keys [id body folder created modified pinned?]}]
  (item/item id
             :note
             (derive-title body)
             {:title (derive-title body)
              :preview (preview-line body)
              :folder folder
              :created created
              :modified modified
              :words (word-count body)
              :pinned pinned?}))

(defn listing->items [entries]
  (mapv entry->item entries))

(def recently-edited
  "The default. Pinned notes are not forced to the top by the sort — that is a
  grouping decision the view makes, and baking it into the comparator would
  mean a user sorting by title still gets pinned notes first, which is not
  what they asked for."
  [[:modified :desc]])

(def default-query
  {:query/sort recently-edited :query/text "" :query/filters []})

(defn pinned [items]
  (vec (filter #(item/attr % :pinned) items)))
