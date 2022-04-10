<script lang="ts" context="module">
  import type { Auth } from "firebase/auth";
  import { writable } from "svelte/store";
  import {
    collection,
    CollectionReference,
    doc,
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

  export function hoursBetween(a: Timestamp, b: Timestamp) {
    return Math.abs(a.toMillis() - b.toMillis()) / 1000 / 3600;
  }

  export function sameDay(a: Date, b: Date) {
    let sameYear = a.getFullYear() == b.getFullYear();
    let sameMonth = a.getMonth() == b.getMonth();
    let sameDate = a.getDate() == b.getDate();
    return sameYear && sameMonth && sameDate;
  }

  export class Week {
    sunday: Date;

    constructor(day = new Date()) {
      this.sunday = new Date(day.getFullYear(), day.getMonth(), day.getDate() - day.getDay());
    }

    day(index: number) {
      let newDay = new Date(this.sunday);
      newDay.setDate(newDay.getDate() + index);
      return newDay;
    }

    get previous() {
      return new Week(this.day(-7));
    }

    get next() {
      return new Week(this.day(7));
    }

    contains(day: Date) {
      let dayMillis = day.getTime();
      return dayMillis >= this.sunday.getTime() && dayMillis < this.next.sunday.getTime();
    }
  }

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

  export class SingleFS<T> {
    collection: CollectionReference<T>;
    document: DocumentReference<T>;
    data: T;

    get id() {
      return this.document.id;
    }

    constructor(firestore: Firestore, collId: string, docId?: string);
    constructor(parent: DocumentReference, collId: string, docId?: string);
    constructor(firestoreOrParent: any, collId: string, docId?: string) {
      this.collection = collection(firestoreOrParent, collId).withConverter({
        toFirestore: (data: T) => data,
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

  export class ListFS<T> extends SingleFS<T> {
    query: Query<T>;
    list: (T & { id: string })[] = [];

    constructor(firestore: Firestore, collId: string, docId?: string);
    constructor(parent: DocumentReference, collId: string, docId?: string);
    constructor(firestoreOrParent: any, collId: string, docId?: string) {
      super(firestoreOrParent, collId, docId);
      this.query = query(this.collection, orderBy("name"));
    }

    async getList() {
      try {
        await getDocs(this.query).then((snapshot) => {
          this.list = snapshot.docs.map((doc) => {
            return { ...doc.data(), id: doc.id };
          });
          this.data = this.list.find((d) => d.id == this.id) as Extract<T & { id: string }, T>;
        });
      } catch (e) {
        this.list = [];
        this.data = null;
        console.error("Caught", e);
      }
      return this.list;
    }

    async getData(id?: string) {
      let searchId = id ?? this.id;
      return this.list?.find((d) => d.id == searchId) ?? (await super.getData());
    }

    update(data: UpdateData<T>, id?: string) {
      if (!id) return super.update(data);
      Object.assign(
        this.list.find((i) => i.id == id),
        data
      );
      updateDoc(doc(this.collection, id), data).catch(console.error);
      return this;
    }
  }

  export interface MT {
    loaded: boolean;
    auth: Auth;
    firestore: Firestore;
    user: SingleFS<UserData>;
    team: SingleFS<TeamData>;
    member: ListFS<MemberData>;
    unverified: ListFS<UnverifiedData>;
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
