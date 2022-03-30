<script lang="ts" context="module">
  import type { Auth } from "firebase/auth";
  import { writable } from "svelte/store";
  import {
    collection,
    CollectionReference,
    doc,
    DocumentData,
    DocumentReference,
    Firestore,
    getDoc,
    getDocs,
    orderBy,
    query,
    Query,
    setDoc,
    Timestamp,
    UpdateData,
    updateDoc,
  } from "firebase/firestore/lite";

  export interface Log {
    hours: number;
    start: Timestamp;
  }

  export interface UserData {
    teamId: string;
  }

  export interface TeamData {
    goal: number;
    name: string;
    ownerId: string;
  }

  export interface MemberData {
    lastAction: Timestamp;
    logs: Log[];
    name: string;
    tracking: boolean;
  }

  export interface UnverifiedData {
    name: string;
  }

  export class FSDataSet<T> {
    collection: CollectionReference<T>;
    document: DocumentReference<T>;
    data: T;

    constructor(firestore: Firestore, collId: string, docId?: string);
    constructor(parent: DocumentReference, collId: string, docId?: string);
    constructor(firestoreOrParent: any, collId: string, docId?: string) {
      this.collection = collection(firestoreOrParent, collId).withConverter({
        toFirestore: (data: T) => data as DocumentData,
        fromFirestore: (snapshot) => snapshot.data() as T,
      });
      if (docId) this.document = docId == "." ? doc(this.collection) : doc(this.collection, docId);
    }

    async getData() {
      if (this.document) {
        await getDoc(this.document)
          .then((snapshot) => (this.data = snapshot.data()))
          .catch(console.error);
      }
      return this.data;
    }

    set(newData: T) {
      if (!this.document) return;
      setDoc(this.document, (this.data = newData)).catch(console.error);
      return this;
    }

    update(data: UpdateData<T>) {
      if (!this.document) return;
      updateDoc(this.document, Object.assign(this.data, data)).catch(console.error);
      return this;
    }
  }

  export class FSListSet<T> extends FSDataSet<T> {
    query: Query<T>;
    list: (T & { id: string })[] = [];

    constructor(firestore: Firestore, collId: string, docId?: string);
    constructor(parent: DocumentReference, collId: string, docId?: string);
    constructor(firestoreOrParent: any, collId: string, docId?: string) {
      super(firestoreOrParent, collId, docId);
      this.query = query(this.collection, orderBy("name"));
    }

    async getList() {
      await getDocs(this.query).then((snapshot) => {
        this.list = [];
        snapshot.forEach((doc) => this.list.push({ ...doc.data(), id: doc.id }));
      });
      return this.list;
    }
  }

  export interface MT {
    loaded: boolean;
    auth: Auth;
    firestore: Firestore;
    user: FSDataSet<UserData>;
    team: FSDataSet<TeamData>;
    member: FSListSet<MemberData>;
    unverified: FSListSet<UnverifiedData>;
  }

  export const mt = writable<MT>({
    loaded: false,
    auth: null,
    firestore: null,
    user: null,
    team: null,
    member: null,
    unverified: null,
  });
</script>
