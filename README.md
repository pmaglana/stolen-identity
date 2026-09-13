# The Stolen Identity
## Scenario

Reconstructed a five-stage OAuth consent-phishing kill chain in a live Azure tenant through forensic analysis of two linked app registrations.

## Environment 

Live multi-user Azure training tenant, Reader Access.

## Investigation

1. ENTRY. A user was phished, completed MFA, and had the resulting session token stolen. That token carried an MFA-satisfied claim, so it sailed past Conditional Access. That user was also, through years of drift, still an Owner on a legacy connector app.

   <img width="725" height="593" alt="1-entry1" src="https://github.com/user-attachments/assets/2b3ee0e0-1d8a-430e-9094-b72a51a0b61f" />
   <img width="895" height="508" alt="1-entry2" src="https://github.com/user-attachments/assets/e46b1f83-555c-4735-a729-ae37e2a5d55d" /></br>


2. ESCALATE. Using those Owner rights, the attacker minted a new client secret on the legacy app. That secret let them authenticate through the client credentials flow as the service principal itself, inheriting the app's directory permissions without ever signing in as a human again. Note the expiry date: set nearly a century out.

   <img width="778" height="642" alt="2-escalate" src="https://github.com/user-attachments/assets/e21b30f3-d01e-4dbe-95f3-e3fb4a06127f" /></br>

3. PIVOT. A single secret dies when it gets rotated. So the attacker registered their own app (every standard user can do this by default in Entra) and added its service principal to the legacy app's Owners list. Now they can re-credential the legacy app forever, even after the first secret is caught.

   <img width="917" height="798" alt="3-pivot2" src="https://github.com/user-attachments/assets/3c5f62a8-2acf-4d5f-a590-3fd73c967969" />
   <img width="727" height="626" alt="3-pivot1" src="https://github.com/user-attachments/assets/ef1984b6-80f3-49ec-9f14-98cebec26a93" /></br>

4. PERSIST. Then the backup plan: a custom scope published on the legacy app's Expose an API blade. This turns the legacy app into a callable backend resource, which means the attacker's own app can request delegated access to it.

   <img width="1217" height="733" alt="4-persist" src="https://github.com/user-attachments/assets/be6e2f4b-18e8-4b67-bf59-d0ffaa928f9f" /></br>

5. LOOT. Finally, a redirect URI on the rogue app pointing at attacker-controlled infrastructure. Combining the rogue app's client ID, that redirect URI, and the exposed API scope produces a working phishing URL. A victim who is already signed in on a corporate device clicks Accept on a consent prompt, and the authorization code lands on the attacker's server.
   

   <img width="1407" height="653" alt="5-loot" src="https://github.com/user-attachments/assets/ad423a3a-becb-4cc4-9c25-8ee91faa61f2" /></br>

## What broke / what surprised me
<!---
The most credible section in the document. Dead ends, wrong guesses, the thing that took an hour. Employers know real work is messy. This section separates you from certificate collectors.
--->
Even if everything was configured, the policy is present and was actively evaluating resources, with just one small slip up such as the this could cause major issues.
## Findings and recommendations
<!---
What you determined, plus 2 or 3 recommendations as if you were reporting to the resource owner.
--->
The deployment was still allowed despite being flagged as non-compliant this is because the resource group policy was incorrectly configured. The policy parameter value should be change from Audit to Deny for an effective naming requirements.

## What I learned
<!---
3 to 5 bullets. At least one technical, one "what I'd do differently."
--->
A policy being assigned does not necessarily mean it is enforcing compliance. Always verify these policy assignments and parameters.
