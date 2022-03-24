<script lang="ts" context="module">
  import { writable } from "svelte/store";
  import type { Auth } from "firebase/auth";
  import type {
    CollectionReference,
    DocumentReference,
    FirestoreDataConverter,
    Timestamp,
  } from "firebase/firestore/lite";

  export interface Log {
    hours: number;
    start: Timestamp;
  }

  export interface UserData {
    id: string;
    teamId: string;
  }

  export interface TeamData {
    id: string;
    name: string;
    ownerId: string;
  }

  export interface TeamPrivateData {
    goal: number;
  }

  export interface MemberData {
    id: string;
    logs: Log[];
    name: string;
    tracking: boolean;
  }

  export interface UnverifiedData {
    id: string;
    name: string;
  }

  export interface ConvertData {
    user: FirestoreDataConverter<UserData>;
    team: FirestoreDataConverter<TeamData>;
    teamPrivate: FirestoreDataConverter<TeamPrivateData>;
    member: FirestoreDataConverter<MemberData>;
    unverified: FirestoreDataConverter<UnverifiedData>;
  }

  export const convertData: ConvertData = {
    user: {
      toFirestore: (user: UserData) => {
        return {
          teamId: user.teamId,
        };
      },
      fromFirestore: (snapshot) => {
        const user = snapshot.data();
        return {
          id: snapshot.id,
          teamId: user.teamId,
        };
      },
    },
    team: {
      toFirestore: (team: TeamData) => {
        return {
          name: team.name,
          ownerId: team.ownerId,
        };
      },
      fromFirestore: (snapshot) => {
        const team = snapshot.data();
        return {
          id: snapshot.id,
          name: team.name,
          ownerId: team.ownerId,
        };
      },
    },
    teamPrivate: {
      toFirestore: (team: TeamPrivateData) => {
        return {
          goal: team.goal,
        };
      },
      fromFirestore: (snapshot) => {
        const team = snapshot.data();
        return {
          goal: team.goal,
        };
      },
    },
    member: {
      toFirestore: (member: MemberData) => {
        return {
          logs: member.logs,
          name: member.name,
          tracking: member.tracking,
        };
      },
      fromFirestore: (snapshot) => {
        const member = snapshot.data();
        return {
          id: snapshot.id,
          logs: member.logs,
          name: member.name,
          tracking: member.tracking,
        };
      },
    },
    unverified: {
      toFirestore: (unverified: UnverifiedData) => {
        return {
          name: unverified.name,
        };
      },
      fromFirestore: (snapshot) => {
        const unverified = snapshot.data();
        return {
          id: snapshot.id,
          name: unverified.name,
        };
      },
    },
  };

  export interface MT {
    loaded: boolean;
    user: {
      data: UserData;
      document: DocumentReference<UserData>;
      collection: CollectionReference<UserData>;
    };
    team: {
      data: TeamData;
      document: DocumentReference<TeamData>;
      collection: CollectionReference<TeamData>;
    };
    teamPrivate: {
      data: TeamPrivateData;
      document: DocumentReference<TeamPrivateData>;
    };
    member: {
      data: MemberData;
      document: DocumentReference<MemberData>;
      collection: CollectionReference<MemberData>;
    };
    unverified: {
      data: UnverifiedData;
      document: DocumentReference<UnverifiedData>;
      collection: CollectionReference<UnverifiedData>;
    };
    cachedMembers: MemberData[];
    selectedMembers: string[];
    cachedUnverifieds: UnverifiedData[];
    selectedUnverifieds: string[];
  }

  export const mt = writable<MT>({
    loaded: false,
    user: {
      data: null,
      document: null,
      collection: null,
    },
    team: {
      data: null,
      document: null,
      collection: null,
    },
    teamPrivate: {
      data: null,
      document: null,
    },
    member: {
      data: null,
      document: null,
      collection: null,
    },
    unverified: {
      data: null,
      document: null,
      collection: null,
    },
    cachedMembers: [],
    selectedMembers: [],
    cachedUnverifieds: [],
    selectedUnverifieds: [],
  });
</script>
