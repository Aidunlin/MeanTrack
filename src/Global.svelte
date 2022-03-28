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
    FirestoreDataConverter,
    getDoc,
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
    private _data: T;

    public get data(): T {
      return this._data;
    }

    public set data(newData: T) {
      if (this.document) {
        setDoc(this.document, (this._data = newData)).catch(console.error);
      }
    }

    private converter<T>(): FirestoreDataConverter<T> {
      return {
        toFirestore: (data: T) => data as DocumentData,
        fromFirestore: (snapshot) => snapshot.data() as T,
      };
    }

    constructor(firestore: Firestore, collId: string, docId?: string);
    constructor(parent: DocumentReference, collId: string, docId?: string);
    constructor(firestoreOrParent: any, collId: string, docId?: string) {
      this.collection = collection(firestoreOrParent, collId).withConverter(this.converter<T>());
      if (docId) {
        if (docId == ".") {
          this.document = doc(this.collection);
        } else {
          this.document = doc(this.collection, docId);
        }
      }
    }

    async getData() {
      if (this.document) {
        await getDoc(this.document)
          .then((snapshot) => (this._data = snapshot.data()))
          .catch(console.error);
      }
      return this._data;
    }

    async updateData(data: UpdateData<T>) {
      if (this.document) {
        await updateDoc(this.document, Object.assign(this._data, data)).catch(console.error);
      }
    }
  }

  export interface MT {
    loaded: boolean;
    auth: Auth;
    firestore: Firestore;
    user: FSDataSet<UserData>;
    team: FSDataSet<TeamData>;
    member: FSDataSet<MemberData>;
    unverified: FSDataSet<UnverifiedData>;
    cachedMembers: (MemberData & { id: string })[];
    cachedUnverifieds: (UnverifiedData & { id: string })[];
  }

  export const mt = writable<MT>({
    loaded: false,
    auth: null,
    firestore: null,
    user: null,
    team: null,
    member: null,
    unverified: null,
    cachedMembers: [],
    cachedUnverifieds: [],
  });
</script>
