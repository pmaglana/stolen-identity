<img width="1976" height="510" alt="banner-stolenidentity2" src="https://github.com/user-attachments/assets/ba04b2b4-1764-4069-b522-2c973954060c" />

<!--- 
## Overview  
An employee got phished through a convincing fake login page, resulting in their password being compromised and MFA was successfully completed, the employee remained the Owner of an enterprise app. The objective is to identify, investigate, and recover evidence related to the incident.

## Objective
To reconstruct the attacker’s actions step by step using only the access and information available in Azure. The investigation will trace the evidence left behind on the app registration identified by the security team as having been compromised.  --->

## Scenario
Reconstructed a five-stage OAuth consent-phishing kill chain in a live Azure tenant through forensic analysis of two linked app registrations.

## Environment
Live multi-user Azure training tenant, Reader access.

## Investigation

#### Objective 1: ENTRY </br>
A user was phished, completed MFA, and had the resulting session token stolen. That token carried an MFA-satisfied claim, so it sailed past Conditional Access. That user was also, through years of drift, still an Owner on a legacy connector app. </br>

Location: Branding & properties</br>
Class of Finding: Internal notes value</br>

   <img width="895" height="508" alt="1-entry2" src="https://github.com/user-attachments/assets/e46b1f83-555c-4735-a729-ae37e2a5d55d" /></br>

#### </br>Objective 2: ESCALATE</br>
Using those Owner rights, the attacker minted a new client secret on the legacy app. That secret let them authenticate through the client credentials flow as the service principal itself, inheriting the app's directory permissions without ever signing in as a human again. Note the expiry date: set nearly a century out.</br>

Location: Certificates & secrets</br>
Class of Finding: A Secret that's dated nearly a century out. </br>

   <img width="778" height="642" alt="2-escalate" src="https://github.com/user-attachments/assets/6e3f7eae-a30d-4edb-af64-a61518b99c69" /></br>

#### </br>Objective 3: PIVOT</br>
A single secret dies when it gets rotated. So the attacker registered their own app (every standard user can do this by default in Entra) and added its service principal to the legacy app's Owners list. Now they can re-credential the legacy app forever, even after the first secret is caught.</br>

Location: Rogue app's Branding & properties</br>
Class of Finding: Internal notes value</br>

   <img width="727" height="626" alt="3-pivot1" src="https://github.com/user-attachments/assets/ef1984b6-80f3-49ec-9f14-98cebec26a93" />
   <img width="917" height="798" alt="3-pivot2" src="https://github.com/user-attachments/assets/3c5f62a8-2acf-4d5f-a590-3fd73c967969" /></br>

#### </br>Objective 4: PERSIST</br>
Then the backup plan: a custom scope published on the legacy app's Expose an API blade. This turns the legacy app into a callable backend resource, which means the attacker's own app can request delegated access to it.</br>

Location: Expose an API</br>
Class of Finding: User consent display name value</br>

   <img width="1217" height="733" alt="4-persist" src="https://github.com/user-attachments/assets/be6e2f4b-18e8-4b67-bf59-d0ffaa928f9f" /></br>

#### </br>Objective 5: LOOT
Finally, a redirect URI on the rogue app pointing at attacker-controlled infrastructure. Combining the rogue app's client ID, that redirect URI, and the exposed API scope produces a working phishing URL. A victim who is already signed in on a corporate device clicks Accept on a consent prompt, and the authorization code lands on the attacker's server.</br>

Location: Rogue app's Redirect URIs</br>
Class of Finding: Suspicious Redirect URI </br>

   <img width="1407" height="653" alt="5-loot" src="https://github.com/user-attachments/assets/ad423a3a-becb-4cc4-9c25-8ee91faa61f2" /></br>
   
This incident is an example of a confused deputy attack because the attacker abused a legitimate, trusted application to access resources using permissions that had already been granted to that application. Rather than directly gaining those privileges, the attacker manipulated the application's existing trust, credentials, and permissions to act on their behalf. This demonstrates how a compromised application can become an intermediary that performs actions the attacker would not normally be authorized to perform.</br>

## What broke / what surprised me

// finalizing. gets reshipped later today.
<!---Any standard user can register an app by default, and that owning an app registration is effectively an unlogged privilege path that a review of Global Admins would completely miss.</br> --->

## Findings and recommendations

Based on the findings identified during the investigation, the following remediation actions are recommended to contain the compromised application and reduce the risk of continued unauthorized access:</br>

   1. Revoke the compromised client secret</br>
   2. Remove the rogue service principal from Owners</br>
   3. Delete the custom exposed API scope</br>
   4. Revoke the OAuth2PermissionGrant explicitly, because containment does not remove it</br>
   5. Remove the attacker redirect URI</br>
   6. Review and reduce the Graph application permissions</br>
   7. Disable default user app registration</br>
   8. Audit every app registration's Owners list the same way you audit directory role membership</br>
   9. Alert on new client secrets and new redirect URIs</br>

## What I learned

// editing. gets reshipped later today.
<!---
My key takeaway from this investigation is that applications should be treated as security sensitive resources. Monitoring roles alone is not sufficient; application ownership, credentials, permissions, and authorization grants must also be regularly reviewed as part of an organization's security strategy.
