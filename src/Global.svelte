<script lang="ts" context="module">
  import type { FirebaseApp } from "firebase/app";
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
    name: string;
    ownerId: string;
  }

  export interface TeamPrivateData {
    goal: number;
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
    public set data(d: T) {
      if (this.document) {
        this._data = d;
        setDoc(this.document, this._data).catch(console.error);
      }
    }

    private getConverter<T>(): FirestoreDataConverter<T> {
      return {
        toFirestore: (data: T) => data as DocumentData,
        fromFirestore: (snapshot) => snapshot.data() as T,
      };
    }

    constructor(firestore: Firestore, collId: string, docId?: string);
    constructor(parent: DocumentReference, collId: string, docId?: string);
    constructor(firestoreOrParent: any, collId: string, docId?: string) {
      this.collection = collection(firestoreOrParent, collId).withConverter(this.getConverter<T>());
      if (docId) {
        this.document = doc(this.collection, docId);
      }
    }

    /** Gets the latest data from Firestore */
    async refreshData() {
      if (this.document) {
        await getDoc(this.document)
          .then(async (snapshot) => {
            this._data = snapshot.data();
          })
          .catch(console.error);
      }
      return this._data;
    }

    /** Updates part/all of data and corresponding Firestore document */
    async updateData(data: UpdateData<T>) {
      if (this.document) {
        Object.assign(this._data, data);
        await updateDoc(this.document, data).catch(console.error);
      }
    }
  }

  export interface MT {
    loaded: boolean;
    firebase: {
      app: FirebaseApp;
      auth: Auth;
      firestore: Firestore;
    };
    user: FSDataSet<UserData>;
    team: FSDataSet<TeamData>;
    teamPrivate: FSDataSet<TeamPrivateData>;
    member: FSDataSet<MemberData>;
    unverified: FSDataSet<UnverifiedData>;
    cachedMembers: (MemberData & { id: string })[];
    selectedMembers: string[];
    cachedUnverifieds: (UnverifiedData & { id: string })[];
    selectedUnverifieds: string[];
  }

  export const mt = writable<MT>({
    loaded: false,
    firebase: {
      app: null,
      auth: null,
      firestore: null,
    },
    user: null,
    team: null,
    teamPrivate: null,
    member: null,
    unverified: null,
    cachedMembers: [],
    selectedMembers: [],
    cachedUnverifieds: [],
    selectedUnverifieds: [],
  });
</script>
