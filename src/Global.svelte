<script lang="ts" context="module">
  import type { Auth } from "firebase/auth";
  import type { CollectionReference, DocumentReference, FirestoreDataConverter, Timestamp } from "firebase/firestore";
  import { writable } from "svelte/store";

  export interface UserData {
    id: string;
    teamId: string;
  }

  export interface TeamData {
    id: string;
    name: string;
    number: number;
    ownerId: string;
  }

  export interface TeamPrivateData {
    goal: number;
  }

  export interface Log {
    hours: number;
    start: Timestamp;
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

  export interface MT {
    auth: Auth;
    user: {
      data: UserData;
      document: DocumentReference<UserData>;
      collection: CollectionReference<UserData>;
    };
    team: {
      data: TeamData;
      document: DocumentReference<TeamData>;
      collection: CollectionReference<TeamData>;
      private: {
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
    };
  }

  export const mt = writable<MT>({
    auth: null,
    user: {
      data: null,
      document: null,
      collection: null,
    },
    team: {
      data: null,
      document: null,
      collection: null,
      private: {
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
    },
  });

  export interface MemberManagement {
    verifiedMembers: MemberData[];
    selectedVerifiedIds: string[];
    unverifiedMembers: UnverifiedData[];
    selectedUnverifiedIds: string[];
  }

  export const memberManagement = writable<MemberManagement>({
    verifiedMembers: [],
    selectedVerifiedIds: [],
    unverifiedMembers: [],
    selectedUnverifiedIds: [],
  });

  export const convertUserData: FirestoreDataConverter<UserData> = {
    toFirestore: (user: UserData) => {
      return {
        teamId: user.teamId,
      };
    },
    fromFirestore: (snapshot, options) => {
      const user = snapshot.data(options);
      return {
        id: snapshot.id,
        teamId: user.teamId,
      };
    },
  };

  export const convertTeamData: FirestoreDataConverter<TeamData> = {
    toFirestore: (team: TeamData) => {
      return {
        name: team.name,
        number: team.number,
        ownerId: team.ownerId,
      };
    },
    fromFirestore: (snapshot, options) => {
      const team = snapshot.data(options);
      return {
        id: snapshot.id,
        name: team.name,
        number: team.number,
        ownerId: team.ownerId,
      };
    },
  };

  export const convertTeamPrivateData: FirestoreDataConverter<TeamPrivateData> = {
    toFirestore: (team: TeamPrivateData) => {
      return {
        goal: team.goal,
      };
    },
    fromFirestore: (snapshot, options) => {
      const team = snapshot.data(options);
      return {
        goal: team.goal,
      };
    },
  };

  export const convertMemberData: FirestoreDataConverter<MemberData> = {
    toFirestore: (member: MemberData) => {
      return {
        logs: member.logs,
        name: member.name,
        tracking: member.tracking,
      };
    },
    fromFirestore: (snapshot, options) => {
      const member = snapshot.data(options);
      return {
        id: snapshot.id,
        logs: member.logs,
        name: member.name,
        tracking: member.tracking,
      };
    },
  };

  export const convertUnverifiedData: FirestoreDataConverter<UnverifiedData> = {
    toFirestore: (unverified: UnverifiedData) => {
      return {
        name: unverified.name,
      };
    },
    fromFirestore: (snapshot, options) => {
      const unverified = snapshot.data(options);
      return {
        id: snapshot.id,
        name: unverified.name,
      };
    },
  };

  export const days = [
    "Sunday",
    "Monday",
    "Tuesday",
    "Wednesday",
    "Thursday",
    "Friday",
    "Saturday",
  ];
</script>
